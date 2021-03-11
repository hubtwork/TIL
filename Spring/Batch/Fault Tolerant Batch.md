### Fault Tolerant in Spring Batch

>✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
>✔ **OS :  macOS Big Sur 11.1**
>
>✔ **Lang: Kotlin**



#### Background

- DB 에서 선결 데이터를 로드해 외부 API 를 이용해 데이터를 가져와 프로세싱 후 DB 에 저장하는 Batch Job 진행
- 스프링 배치의 `Tasklet` 이 아닌 `Reader` > `Processor` > `Writer` 구조로 작성
- `Processing` 과정 중 API 요청에서 오류가 발생하거나 관련 응답 데이터가 존재하지 않아 오류가 발생할 경우, `Batch` 의 정상적인 실행에 영향을 주지 않아야 함
- 이에 전반적인 설계를 Fault Tolerant 하게 설계하거나 `Skip / Retry` 전략을 통해 예기치 못한 오류가 발생했을 때를 방지함
- 이 글에는 Fault Tolerant 한 배치 설계 방식에 대해 설명하고 `Skip / Retry` 전략은 따로 다루도록 함



#### Fault Tolerant 한 배치 설계

> 배치 어플리케이션의 Fault Tolerant 는 다양한 상황에서 그 상황에 맞게 구현할 수 있다
>
> 필자는 `DB Reader` > `API Request Processer` > `DB Writer` 의 구조로 다단계의 스텝을 이루고 맞물려 작동하는 배치를 작성, 수행하고 있으며 이에 맞춰 예시를 설명한다

- **JPA Entity Manager 에 의한 예외 케이스**
  - **Entity Manager** 를 이용해 *chunk* 단위의 DB Connection 을 진행할 때, 주의할 점은 Entity Manager 가 chunk 단위로 생성 및 전파가 이루어진다는 점이다
  - **Entity Manager** 는 하나의 chunk 에 대한 정보를 정상적으로 처리한 후 다음 chunk 를 로드할 때까지 App Context 에 생존하는데 이를 유념해 `chunk Size` 및 `page Size` 를 일치하게 설계하여 **JPA** 의 영속성 컨텍스트가 소멸하기 전에 정상 처리할 수 있도록 한다
  - 실제로 JPA 기반의 스프링 배치 환경에는 `chunk` , `pageSize` , `fetchSize` 라는 주요 설정값이 존재하는데 문제 발생을 예방하기 위해 각 배치 서비스에 맞게 조절을 통해 테스트를 진행해 적정한 값의 조합을 찾아내는 것 또한 중요하다 ( 일반적으로 `pageSize` , `chunkSize` 는 동일하게 두는 것이 단위를 맞추는 데에 좋다 )
    - `chunk` - 스프링 배치 환경에서 처리하는 데이터의 최소 단위. 
    - `pageSize` - 쿼리를 페이징할 때 한 페이지에 들어가는 데이터의 개수.
    - `fetchSize` - DB 에서 한번에 가져올 데이터의 개수.
  - 만일 영속성 컨텍스트가 이미 소멸되었는데 해당 컨텍스트에서 다루는 객체를 로드할 경우, 해당 객체에 대한 영속성 컨텍스트의 세션이 이미 소멸하였기 때문에 `LazyInitializationException` 과 같은 예외가 발생할 수 있다
- **스텝의 세분화가 이루어지지 않은 케이스**
  - 외부 API 를 이용해 데이터를 가져와 프로세싱하는 경우, 원하는 데이터를 얻기 위해 **동기적인 API 요청** 을 통해 가져와야하는 경우가 있다
  - 실제 배치를 이용해 작업해야 하는 데이터는 `API C` 의 응답인데 이를 위해서 `API A` 의 응답으로 `API B` 의 응답을 요청해야 하는 경우이다
  - 해당 동기적인 요청을 작업하기 위해 `Tasklet` 으로 일괄처리하거나 `One Proccessor` 스타일로 하나의 프로세서에서 모든 동기 요청을 진행 후 데이터를 저장하는 경우가 있는데, 이는 매우 **위험함** 
  - **외부 API 에 의존하는 서비스의 경우** 에러 발생으로 인해 배치가 실패하거나 혹은 API 서버 내의 데이터 말소로 인해 해당 응답 데이터가 사라져 필요할 때 재 조회가 불가할 수 있기 때문에 각 API 응답 `Raw Data` 는 보존할 필요성이 있다
  - 이에 위 예시와 같은 상황에서 아래와 같이 설계하는 것을 예로 들 수 있다
    - `Step 1` - 사전 데이터를 DB 에서 로드, 이에 기반해 API A 에 요청한 후 그 응답을 DB 에 저장
    - `Step 2` - DB 내의 API A response Data 들에 대해 일괄적으로 API B 를 요청 후 그 응답을 DB 에 저장
    - `Step 3` - DB 내의 API B response Data 들에 대해 일괄적으로 API C 를 요청 후 그 응답을 DB 에 저장
    - 각 Response Data 테이블에 Flag ( Boolean 혹은 Datetime 등 ) 을 이용해 중복된 프로세싱을 진행하지 않도록 스텝을 추가하는 등으로 중복 처리를 방지한다면 훌륭한 배치 처리가 될 수 있을 것이다
- **DB Connection 누수 발생 케이스**
  - 배치 진행 로직 상에서 DB Connection 을 오래 홀드하고 있다면 이는 **DB 운영** 에 있어 심각한 문제를 초래할 수 있다
  - `Processor`에서 APi 요청 / 응답을 진행할 때, 해당 요청에 대한 응답이 올바르게 작동하지 않아 대기시간이 길어진다면 이를 해결하기 위해 DB Connection 의 연장이 필요함
  - `JPA` 의 경우 일반적으로 Hikari CP 와 같은 구현체를 사용하며  해당 API response time 이 길어질 경우 각 커넥션 풀의 세션 또한 **불필요한 연장 지속** 이 이루어지기 때문에 외부 API 에 의한 Connection 누수 현상이 발생한다
  - 이를 위해 `chunk hSize` 와 `fectch Size` 의 조합에 대한 최적해를 발행하기 위한 테스트를 진행해 값의 조정을 진행함 ( 일반적으로 `chunkSize` 와 `pageSize` 는 최적해를 가질 때, Connection Leak 를 방지할 수 있음 )
  - `spring.datasource.hikari.leakDetectionThreshold=1000`  과 같이  Hikari Leakpoint 에 대한 대비 및 작업 부여를 통해 정상적으로 해당 애플리케이션의 빌드가 작동할 수 있도록 하는 환경 구성이 중요함



### REFERENCE

- https://docs.spring.io/spring-batch/docs/4.2.x/api/org/springframework/batch/item/database/builder/JpaPagingItemReaderBuilder.html
- https://docs.spring.io/spring-batch/docs/4.2.x/api/index.html