### JPA Repository 관련 정보

> ✔ **DB : MariaDB 10.1**
>
> ✔ **Lang : Kotlin**

#### Repository 기본 구성

~~~kotlin
@Repository				// Jpa Repository Bean DI
interface PipelinedMatchRepository: JpaRepository<PipelinedMatch, Long>
// JpaRepository<[Entity Class], [Id Type Class]> Interface 를 상속받아 정의
~~~

