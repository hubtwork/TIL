### JpaPagingItemReader 사용시 레코드 중복 / 스킵 되는 문제

~~~kotlin
// JpaPagingItemReader 의 일반적 구현체 ( Select 문 활용 )
private fun matchListItemReader() : JpaPagingItemReader<MatchList> {
        var itemReader = JpaPagingItemReader<MatchList>()
        itemReader.setQueryString("select m from matchList m where isScanned = 0")
        itemReader.pageSize = matchChunk
        return itemReader
    }
~~~

- Chunk 단위로 실행시 start / chunkSize 개념의 쿼리가 반복적으로 실행됨.

#### 쿼리 반복 예시

~~~mariadb
# Format
select m
from matchList m
where isScanned = 0
limit [start], [chunkSize]
~~~

~~~mariadb
# 1st read
select m
from matchList m
where isScanned = 0
limit 0, 100
~~~

~~~mariadb
# 2nd read
select m
from matchList m
where isScanned = 0
limit 100, 100
~~~

- 정확한 정렬기준이 정해져있지 않으면 매번 청크 단위로 페이징할 때, 랜덤하게 정렬된 결과가 나오기 때문에 건너뛰어지거나 중복해서 읽히는 데이터가 생길 수 있음.
- 이를 위해 정확한 정렬기준이 있어야함.

~~~mariadb
# Recommended
select m
from matchList m
where isScanned = 0
order by matchId
~~~

---

