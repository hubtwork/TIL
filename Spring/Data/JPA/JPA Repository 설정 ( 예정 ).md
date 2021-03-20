### JPA Repository

> ✔ **DB : MariaDB 10.1**
>
> ✔ **Lang : Kotlin**



#### JPA Entity

- 본 글에서 `JPA Repository` 를 위한 `Entity`
- `Entity` 는 **RDBMS** 의 **Table** 역할

~~~kotlin
// Spring Data - JPA Bean Type Entity 명시
@Entity				
// Table 명, 테이블의 인덱스에 대해 명시
@Table(name = "pipelinedMatch", indexes = [ Index(columnList = "season"), Index(columnList = "queueType") ])
class PipelinedMatch (
// 객체 생성 인자
    matchId: Long,
    season: Int,
    queueType: Int,
  	...
){
		// Primary Key 등록
    @Id
  	// Column Name 명시
    @Column(name = "matchId")
    val matchId: Long = matchId

    @Column(name = "season")
    val season: Int = season

    @Column(name = "queueType")
    val queueType: Int = queueType
  
  	...
}
~~~



#### JPA Repository

~~~kotlin
@Repository				// Repository Bean DI
interface PipelinedMatchRepository: JpaRepository<PipelinedMatch, Long>
// JpaRepository<[Entity Class], [Id Type Class]> Interface 를 상속받아 정의
~~~

- `@Repository` - DAO 클래스임을 명시하는 Annotation. 커스텀 **Repository** Interface 를 스프링 루트 컨테이너를 **Bean** 으로 등록함

- `JpaRepository<T, Id> ` - **JpaEntity T** 에 대한 Repository Interface. `@Repository` Annotation 이 명시되지 않더라도 사용가능하나 해당 인터페이스의 역할을 명시하기 위해 사용함.

- **JPA Repository** 의 기본 기능

  | **Method** | **Functionality**                                |
  | ---------- | ------------------------------------------------ |
  | save       | 데이터 저장 ( Insert, Update 시 사용 )           |
  | findOne    | Id 값 ( Primary Key ) 을 이용해 단일 데이터 조회 |
  | findAll    | 전체 데이터 조회 ( 페이징, 데이터 정렬 가능 )    |
  | count      | 데이터 개수 조회                                 |
  | delete     | 원하는 데이터 삭제                               |



#### JPA Query Method

- `JpaRepository` 는 키워드를 이용해 데이터 조회를 위한 쿼리 함수의 작성을 지원함

- **findBy** - 원하는 Column 에 대한 **조회** 쿼리

- **countBy** - 원하는 Column 에 대한 조회 결과 **데이터 개수** 쿼리

- **List\<T>** 를 반환형으로 할 경우, 리스트 객체로 결과를 받아오며 **T** 를 반환형으로 할 경우 단일 객체로 결과를 받아옴

  ~~~kotlin
  @Repository
  interface PipelinedMatchRepository: JpaRepository<PipelinedMatch, Long> {
    // queueType 컬럼에 대한 조건절 다수 데이터 조회
  	fun findByQueueType(queueType: Int): List<PipelinedMatch>
    // matchId , season 컬럼에 대한 조건절 단일 데이터 조회
    fun findByIdAndSeason(matchId: Long, season: Int): PipelinedMatch?
  }
  ~~~

- **JPA Query Methods**

| Keyword             | Sample                                                       | JPQL snippet                                                 |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `And`               | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is,Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`           | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`          | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`     | `findByAgeLessThanEqual`                                     | `… where x.age ⇐ ?1`                                         |
| `GreaterThan`       | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`  | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`             | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`            | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`            | `findByAgeIsNull`                                            | `… where x.age is null`                                      |
| `IsNotNull,NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`              | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`           | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`      | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`        | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`        | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`           | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`               | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`             | `findByAgeNotIn(Collection<Age> age)`                        | `… where x.age not in ?1`                                    |
| `True`              | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`             | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`        | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |



#### JPA Native Query

- `@Query` Annotation 의 **nativeQuery** 플래그를 통해 DBMS 의 Custom Query 지원

- 이 때, 함수의 파라미터들을 **?n** 의 형태로 n번째 파라미터를 매핑하여 정의할 수 있음

  ~~~kotlin
  @Repository
  interface PipelinedMatchRepository: JpaRepository<PipelinedMatch, Long> {
    // 해당 JPA 는 MariaDB 에 연동하여 사용하므로 MariaDB 의 쿼리 스타일대로 쿼리를 작성할 수 있음
    @Query(value = "SELECT * FROM PipelinedMatch WHERE matchId = ?1", nativeQuery = true)
  	fun getMatchdataWithMatchId(matchId: Int): PipelinedMatch?
  }
  ~~~



### REFERENCE

- https://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/

