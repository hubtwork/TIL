### Kotlin-Jpa Plugin

> ✔ **Module : Spring Data JPA**



#### Background

- 기존에 `Java` 로 `JPA` 를 다루다가 최근 들어 `Kotlin` 으로 작업을 바꿔 진행하였음
- 코틀린은 `Lombok` 의 사용이 불필요할 정도로 편리하고 강력하였으며 이에 따라 `JPA Entity` 의 생성자 관리에 대한 고찰이 필요해짐
- `JPA` 의 **Hibernate** 는 `no-arg constructor` 를 리플렉션함으로서 엔티티를 생성하는데 이에 따른 접근이 필요함



#### Kotlin-JPA

- `kotlin` 에서 **no-arg** 플러그인을 래핑해 제공하는 플러그인
- `@Entity` , `@Embedded`, `@Embeddable` 등의 **Annotation** 을 갖는 클래스에 대해 기본 생성자를 만들어 줌
- **사용** - gradle Query DSL 을 이용해 플러그인 적용

~~~kotlin
plugins {
...
  // Latest Version
	id("org.jetbrains.kotlin.plugin.jpa") version "1.5.0-M2"
...
}
~~~

- **역할** - `no-arg constructor` 의 자동 생성을 통해 위 제약적인 클래스들의 문제사항을 커버해줌



#### Kotlin-Spring

- `Kotlin` 으로 작업을 하다보면 기본 생성자 문제 뿐만 아니라 상속에 관해서도 문제가 발생함
- **이유 )** `Kotlin` 의 class 는 기본적으로 `final` 이어 **Spring** 의 클래스들을 상속해 이용하기 위해서는 `open class` 로 사용하여야 함
- 이를 위해 `@Component` , `@Transactional` , `@Configuration` 등의 **Annotation** 을 갖는 클래스에 대해 자동으로 `all-open` 적용을 지원해줌
- **사용** - gradle Query DSL 을 이용해 플러그인 적용

~~~kotlin
plugins {
...
	// Latest Version
  id("org.jetbrains.kotlin.plugin.spring") version "1.5.0-M2"
...
}
~~~

- **역할** - 스프링의 주요 **Annotation** 들의 `all-open` 을 지원해 관련 클래스 및 메소드들을 `all-open` 처리해줌



### REFERENCE

- http://wiki.sys4u.co.kr/display/SOWIKI/1.+Maven+Configuration+for+Kotlin+Springboot+Project
- https://plugins.gradle.org/plugin/org.jetbrains.kotlin.plugin.jpa

