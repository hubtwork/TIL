### Spring WebClient - Basic

> ✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
> ✔ **Lang: Kotlin**



#### Background

- **Spring 5** 와 **Spring Boot 2.0** 부터 아래 사항에 대해 공지됨
- **RestTemplate**
  - 기존 Spring Web 에서 지원하던 RestTemplate 에 대한 정식적 지원이 종료됨 
  - 더불어 비동기 통신을 지원하던 AsyncRestTemplate 또한 Deprecated
- <span style="color: red">새로운 Web Request Interface</span> 로 **WebClient** 를 제시함



#### WebClient

- **Spring Reactive Web** 에서 지원하는 Web Request Interface

- 리액티브, 논블로킹 방식의 HTTP Request 인터페이스 지원

  - **리액티브** : Streaming + Lightweight + Real-time 의 설계 패러다임
  - **논블로킹** : 데이터 송수신이 없어도 스레드를 블로킹하지 않고 여러 입출력을 처리할 수 있게 하는 방식

- **Dependency**

  - Spring Boot Project - WebFlux

  - ~~~kotlin
    // in build.gradle
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    ~~~

  - ~~~xml
    <!-- in pom.xml -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    ~~~

- **Config**

  - WebClient 는 Builder 패턴을 이용해 Instance 를 생성하고 Bean 으로 등록해서 사용하는 것을 추천함

  - ~~~kotlin
    // Builder 패턴을 이용해 WebClient Instance 를 생성
    @Bean
    fun webClient(builder: WebClient.Builder): WebClient =
      builder
    		.baseUrl("http://localhost:8080")	// BaseUrl 등록
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE) // 헤더 등록
        .build()
    ~~~

  - <span style="color:red">**( 선택 )**</span> **Logger Filter** 

  - Logger 를 이용해 Request 및 Response 결과를 자동으로 로깅할 수 있게 확장

  - ~~~kotlin
    private val logger = LoggerFactory.getLogger(WebClientConfig::class.java)
    
    // ExchangeFilter 의 Request Processor 를 이용해 ClientRequest 클래스의 Request 정보를 로깅
    fun requestLoggerFilter() = ExchangeFilterFunction.ofRequestProcessor {
      logger.info("Logging request: ${it.method()} ${it.url()}")
      Mono.just(it)
    }
    
    // ExchangeFilter 의 Response Processor 를 이용해 ClientResponse 클래스의 Response 정보를 로깅
    fun responseLoggerFilter() = ExchangeFilterFunction.ofResponseProcessor {
      logger.info("Response status code: ${it.statusCode()}")
      Mono.just(it)
    }
    
    @Bean
    fun webClient(builder: WebClient.Builder): WebClient =
      builder
        .filter(requestLoggerFilter())	// request 필터 등록
        .filter(responseLoggerFilter())	// response 필터 등록
    		.baseUrl("http://localhost:8080")	
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE) 
        .build()
    ~~~



#### Request

- **DTO Class**

  - ~~~kotlin
    // Kotlin-based example DTO Data Class
    data class ExampleDTO(
      var uId: Long,
      var id: String,
      var pw: String
    )
    ~~~

- **Example**

  - WebClient 는 <u>Spring Reactor</u> 를 이용해 통신하기 때문에 데이터를 가져올 때 **Mono** 와 **Flux** 로 DTO 를 Wrapping 하여 가져옴

  - ~~~kotlin
    val exampleData: Mono<EampleDTO> = 
    	webClient
    		.get()
    		.uri("/.../...")
    		.retrieve()
    		.bodyToMono(ExampleDTO::class.java)
    ~~~

  - <span style="color:red">**( Synchronous )**</span> 리액티브 스트림이 아닌 RestTemplate 과 같은 <u>블로킹 방식</u> 으로 진행하고 싶다면 **Mono.block() 함수** 와  **Flux.blockFirst()** 를 이용할 수 있음

  - **block** 을 사용할 경우 <span style="color:red">리액티브 파이프라인의 장점이 사라지고</span> 메인스레드에서 모든 호출이 이뤄지기 때문에 **Spring 에서 권하지 않음**

  - ~~~kotlin
    val exampleData: Mono<EampleDTO> = 
    	webClient
    		.get()
    		.uri("/.../...")
    		.retrieve()
    		.bodyToMono(ExampleDTO::class.java)
    		.block()
    ~~~

