### JPA Entity 관련 정보



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





