### JpaPagingItemReader 을 이용해 동일 레코드를 조회, 수정할 시 발생하는 문제



#### Background

- 문제 상황은 아래 MatchList 엔티티에서 matchData 가 null 인 레코드를 모두 가져와 matchId 에 따른 프로세싱 후, 다시 저장하는 배치를 실행하면서 발생함. 

> MatchList 엔티티

~~~kotlin
@Entity
@Table(name = "matchList")
class MatchList (
    matchId: Long
){
    @Id
    @Column(name = "matchId")
    val matchId: Long = matchId

    @Column(name = "matchData", columnDefinition = "Text")
    var matchData: String? = null
}
~~~

> LoadMatchesJobConfiguration 배치 예시 ( 예시를 위해 ChunkSize 및 상세 작업 조정 )

~~~~kotlin
@Configuration
@EnableBatchProcessing
class LoadMatchesJobConfiguration(
  private val jobBuilderFactory: JobBuilderFactory,
  private val stepBuilderFactory: StepBuilderFactory,
  private val entityManagerFactory: EntityManagerFactory,
) {
  companion object {
    const val chunkSize = 2
  }

  @Primary
  @Bean("loadMatchesJob")
  fun loadMatchesJob(): Job =
    jobBuilderFactory.get("loadMatchesJobOnApiJob")
      .start(stepInjectMatchDetail())         
      .build()

  @Bean
  @JobScope
  fun stepInjectMatchDetail() : Step =
    stepBuilderFactory.get("Step_InjectMatchData")
      .chunk<MatchList, MatchList>(chunkSize)
      .reader(matchListForAPIItemReader())
      .processor(getMatchDetailItemProcessor())
      .writer(matchListItemWriter())
      .faultTolerant()
      .build()

  private fun matchListItemReader() : JpaPagingItemReader<MatchList> {
    var itemReader = JpaPagingItemReader<MatchList>()
    itemReader.setEntityManagerFactory(entityManagerFactory)
    itemReader.setQueryString("select m from MatchList m where matchData != null order by matchId")
    itemReader.pageSize = matcherChunk
    return itemReader
  }

  private fun matchScanProcessor() = MatchListItemProcessor()

  private fun matchListItemWriter() : JpaItemWriter<MatchList> {
    val jpaItemWriter: JpaItemWriter<MatchList> = JpaItemWriter()
    jpaItemWriter.setEntityManagerFactory(entityManagerFactory)
    return jpaItemWriter
  }
}

~~~~



#### Problem

> 아래와 같이 데이터가 존재한다고 가정했을 때,

~~~kotlin
MatchList(1, null)
MatchList(2, null)
MatchList(3, null)
MatchList(4, null)
MatchList(5, null)
MatchList(6, null)
MatchList(7, null)
~~~

> 첫 번째 ItemRead

~~~
MatchList(1, null)
MatchList(2, null)
~~~

> 두 번째 ItemRead

~~~
MatchList(5, null)
MatchList(6, null)
~~~

- 위와 같이 두개의 레코드를 건너 뛰는 현상이 발생함 !



#### Reason

- JpaPagingItemReader 구현체 확인

~~~java
public class JpaPagingItemReader<T> extends AbstractPagingItemReader<T> {
  ...
  @Override
	@SuppressWarnings("unchecked")
	protected void doReadPage() {
		...
		Query query = createQuery().setFirstResult(getPage() * getPageSize()).setMaxResults(getPageSize());
		...
	}
  ...
}
~~~

- 상속받은 AbstractPagingItemReader 구현체 확인

~~~java
public abstract class AbstractPagingItemReader<T> extends AbstractItemCountingItemStreamItemReader<T> 
        implements InitializingBean {
  ...
  private volatile int page = 0;
  ...
  public int getPage() {
		return page;
	}
  ...
  @Nullable
	@Override
	protected T doRead() throws Exception {
		synchronized (lock) {
				...
				doReadPage();
				page++;
        ...
	}
	...
}
~~~

- 같은 엔티티에 대해 Process, Write 한 후 다시 Read 할 때 변경된 부분을 제외한 나머지 레코드가 조회되며 page 는 증가한 채로 적용됨
- 예시는 아래와 같음

> 1번째 Read ( page = 0, pageSize = 2 )

~~~
MatchList(1, null)		// Read
MatchList(2, null)		// Read
MatchList(3, null)
MatchList(4, null)
MatchList(5, null)
MatchList(6, null)
MatchList(7, null)
~~~

> 1번째 Chunk 작업 후 상태

~~~
MatchList(1, "...")		// Write
MatchList(2, "...")		// Write
MatchList(3, null)
MatchList(4, null)
MatchList(5, null)
MatchList(6, null)
MatchList(7, null)
~~~

> 2번째 Read ( page = 1, pageSize = 2 )

~~~
MatchList(3, null)
MatchList(4, null)
MatchList(5, null)		// Read
MatchList(6, null)		// Read
MatchList(7, null)
~~~



#### Solution

- JpaPagingItemReader 를 상속해 page 초기화

~~~kotlin
class ModifyingJpaPagingItemReader<T> : JpaPagingItemReader<T>() {
    // 매 Page 를 0 으로 초기화하여 다시 Read 할 때, 문제 없도록 확인
    override fun getPage(): Int {
        return 0
    }
}
~~~

- 위와 같이 변경하여 활용하면, Chunk 단위 Read 시 page 가 0 으로 초기화 되기 때문에 pageSize 만큼 레코드를 건너뛰는 문제 해결

~~~kotlin
// ItemReader 활용 예시
...
  private fun matchListItemReader() : ModifyingJpaPagingItemReader<MatchList> {
    var itemReader = ModifyingJpaPagingItemReader<MatchList>()
    itemReader.setEntityManagerFactory(entityManagerFactory)
    itemReader.setQueryString("select m from MatchList m where matchData != null order by matchId")
    itemReader.pageSize = matcherChunk
    return itemReader
  }
...
~~~

---

