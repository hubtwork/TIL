### JPA Entity 관련 정보

> ✔ **DB : MariaDB 10.1**
>
> ✔ **Lang : Kotlin**

#### Entity 기본 구성

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
    matchEndTime: Long,
    matchData: String
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

    @Column(name = "matchEndTime")
    val matchEndTime: Long = matchEndTime
		// Column 타입 명시 지정 가능
    @Column(name = "matchData", columnDefinition = "Text")
    val matchData: String = matchData

}
~~~

- 테이블명, 컬럼명에 대한 명시는 <span style="color: red">데이터베이스의 테이블을 하나의 Object</span> 로서 보는 JPA 생태계에서 중요함 ( 엔티티 클래스를 DB 에 대한 커서이자 DTO 로서 활용성을 극대화하는 것이 포인트 )
- 테이블의 성능을 위해 **인덱스 컬럼**에 대한 명시를 통해 타겟 RDBMS 에 전달해줌
- **String 변수**의 경우 기본적으로 <u>varchar 형</u> 으로 변환하기 때문에 대용량의 스트링 ( ex_ JsonString ) 등의 보관이 필요하다면 Text 로 명시해주는 것이 좋음 ( MariaDB의 경우 )
- 엔티티 Bean 이 주입되었음이 확인되면 JPA 는 해당 DB 에 엔티티가 존재하지않거나, 구성이 다를 시 **Hibernate DDL** 을 통해 생성 / 변경함



#### 복함키 Entity 구성

~~~kotlin
// Serializable 상속받아 복합키로 사용할 칼럼 명시
class ComposableKey(
    val accountId: String = "",
    val matchId: Long = 0
): Serializable

@Entity
@Table(name = "search_usermatch")
// 복합키 사용시 IdClass Annotation 이용해 해당 클래스 주입
@IdClass(ComposableKey::class)
class UserWithMatch (
  accountId: String,
  matchId: Long
){
  // 복합키 형태의 Entity의 경우, 해당하는 칼럼들 모두에 Id Annotation 명시
  @Id
  // Mysql / MariaDB 의 Varchar 타입의 경우, length 파라미터를 통해 길이 지정
  @Column(name="accountId", length = 100)
  var accountId: String = accountId

  @Id
  @Column(name="matchId")
  var matchId: Long = matchId
}
~~~





