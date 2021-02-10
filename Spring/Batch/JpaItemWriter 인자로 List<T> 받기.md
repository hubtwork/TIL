### Processor 에서 LIST\<T> Return 후 JpaItemWriter Write ?

~~~java
// JpaItemWriter 의 write 메소드
@Override
	public void write(List<? extends T> items) {
		EntityManager entityManager = EntityManagerFactoryUtils.getTransactionalEntityManager(entityManagerFactory);
		if (entityManager == null) {
			throw new DataAccessResourceFailureException("Unable to obtain a transactional EntityManager");
		}
		doWrite(entityManager, items);
		entityManager.flush();
	}

~~~

- 만약 외부에서 List\<T> 형식의 데이터가 들어오면, doWrite 메소드 실행시 "T is Entity , But List\<T> is Not Entity" 의 상황이 발생, doWrite 진행시 Error 가 발생.
- 이를 해소하기 위해 "List\<T> 를 T 로 바꾸어 하나의 List 로 전달" 하는 전략을 채택.

~~~kotlin
class JpaListItemWriter<T>(
    jpaItemWriter: JpaItemWriter<T>
)
    : JpaItemWriter<List<T>>()
{
    private val jpaItemWriter = jpaItemWriter

    override fun write(items: MutableList<out List<T>>) {
        // LOGIC
        // 리스트 아이템을 모두 Processor 로 부터 전달 받은 후 "중복제거" 후 병합하여 Writing
        var allItems = mutableSetOf<T>()
        items.forEach { allItems.addAll(it) }
        jpaItemWriter.write(allItems.toList())
    }
}
~~~

- List\<T> 를 받았을 때, T 로 인지할 수 있도록 <span style='color:red'>JpaItemWriter<List\<T>></span> 을 상속받고, write 메소드만 수정.
- List\<T> 데이터들을 모두 가져와 중복제거 ( Set 활용 ) 진행 후 병합하여 JpaItemWriter\<T> 의 write 메소드 실행.

##### 활용 예시

~~~kotlin
fun writerMatchesToScan() : JpaListItemWriter<MatchList> {
        val jpaItemWriter: JpaItemWriter<MatchList> = JpaItemWriter()
        jpaItemWriter.setEntityManagerFactory(entityManagerFactory)
        return JpaListItemWriter(jpaItemWriter)
    }
~~~
