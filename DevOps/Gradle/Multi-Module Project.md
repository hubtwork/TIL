### Gradle Multi Module

>✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
>✔ **Lang: Kotlin**



#### Background

- Spring Boot 를 이용한 모듈들로 이루어진 프로젝트를 진행중
- 여러 모듈 들에서 공통적으로 활용되는 기능들을 정의한 공통 모듈을 작성
- 각 Spring Boot Application 을 배포할 때 의존 모듈을 사용할 수 있게 빌드해야함



#### Project Structure

- 해당 프로젝트의 Gradle 스크립트는 Kotlin DSL 로 작성됨
- Kotlin DSL 이 아닌 경우 kts 확장자를 제거, 문법에 맞게 변경
- 본 멀티 모듈 프로젝트는 루트 프로젝트의 gradle 로 전체 프로젝트를 관리

~~~
Katarina ( root )
  - gradle
  - Katarina-core
     └ src
  - Katarina-batch
     └ src
  - Katarina-api
  	└ src
  - build.gradle.kts
  - setting.gradle.kts
  ...
~~~

##### build.gradle.kts

- kotlin DSL 은 기존 

~~~groovy
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile
import org.springframework.boot.gradle.tasks.bundling.BootJar

plugins {
    kotlin("jvm") version "1.4.21"
  	// 스프링 플러그인 ADD
    kotlin("plugin.spring") version "1.4.21"
    kotlin("plugin.jpa") version "1.4.21"
    id("org.springframework.boot") version "2.4.2"
    id("io.spring.dependency-management") version "1.0.11.RELEASE"
}
// 전체 프로젝트에 대한 공통 영역
allprojects {
// Dependency repository 정의
    repositories {
        mavenCentral()
    }
}
// 당 프로젝트에서 관리할 모든 모듈들의 공통 영역
subprojects {
// 참조된 플러그인 적용
    apply(plugin="kotlin")
    apply(plugin="org.springframework.boot")
    apply(plugin="io.spring.dependency-management")
    apply(plugin="org.jetbrains.kotlin.plugin.spring")

    group = "com.hubtwork"
// 모든 프로젝트에 대해 해당 dependency가 적용됨
    dependencies {
        // jpa
        implementation("org.springframework.boot:spring-boot-starter-data-jpa")
        runtimeOnly("com.h2database:h2")
        runtimeOnly("org.mariadb.jdbc:mariadb-java-client")
        // webflux
        implementation("org.springframework.boot:spring-boot-starter-webflux")
        // reactor
        implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
        // jackson
        implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
        // Gson
        implementation("com.google.code.gson:gson:2.8.6")
    }

    tasks {
        compileKotlin {
            kotlinOptions {
                freeCompilerArgs = listOf("-Xjsr305=strict")
                jvmTarget = "1.8"
            }
            dependsOn(processResources)
        }
        "test"(Test::class) {
            useJUnitPlatform()
        }
    }

}
// 하부 모듈 1 ( 공통 모듈 )
project(":katarina-core") {
    val bootJar: BootJar by tasks
    val jar: Jar by tasks
// 공통 모듈의 경우, SpringBoot Application 으로 실행하지 않고 라이브러리로서 쓰기 때문에 bootJar 는 disable
// Spring Boot 에서 bootRepackage > bootJar 로 변경되면서 jar 를 enable 하지 않으면 빌드 되지 않는 이슈가 발생
    bootJar.enabled = false
    jar.enabled = true

    version = "1.0-SNAPSHOT"
// 해당 모듈에서만 사용할 의존성 정의
    dependencies {
// Test
        testImplementation("org.springframework.boot:spring-boot-starter-test")
    }

}
// 하부 모듈 2
project(":katarina-batch") {
    apply(plugin="org.jetbrains.kotlin.plugin.jpa")

    version = "1.0.A"
// 해당 모듈에서만 사용할 의존성 정의
    dependencies {
        implementation(project(":katarina-core"))
        // Spring Batch
        implementation("org.springframework.boot:spring-boot-starter-batch")
        testImplementation("org.springframework.batch:spring-batch-test")
        // Test
        testImplementation("org.springframework.boot:spring-boot-starter-test")
    }

}
// 하부 모듈 3
project(":katarina-api") {

    version = "1.0-SNAPSHOT"
// 해당 모듈에서만 사용할 의존성 정의
    dependencies {
        implementation(project(":katarina-core"))
    }
}
~~~

- setting.gradle.kts

~~~kotlin
// root 프로젝트 설정
rootProject.name = "katarina"
// 포함할 모듈 추가
include("katarina-core")
include("katarina-batch")
include("katarina-api")
~~~



#### Build Command

~~~shell
# 루트 디렉토리로 이동 
$ cd /.../Katarina
# gradle 이용해 원하는 모듈, bootJar 로 빌드 태스크 진행
$ gradle :katarina-batch:bootJar
# 빌드된 jar 파일 실행
# 해당 스프링 부트 어플리케이션에서 공통 모듈을 사용하는 Bean 들이 모두 정상 작동함을 확인할 수 있음
$ java -jar ./katarina-batch/build/libs/katarina-batch-1.0.A.jar
~~~