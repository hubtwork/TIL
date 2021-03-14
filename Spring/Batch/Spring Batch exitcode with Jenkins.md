### Spring Batch exitcode with Jenkins

>✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
>✔ **OS :  macOS Big Sur 11.1**
>
>✔ **Lang: Kotlin**



 ~~~shell
2021-03-07 01:38:05.226  INFO 115345 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=loadMatches]] completed with the following parameters: [{buildNum=97}] and the following status: [FAILED] in 2m10s816ms

Process finished with exit code 0
 ~~~

- **Jenkins** 를 통해 **Spring Batch** 를 빌드, 실행했을 때 배치 실행중 문제가 발생하면 위와 같이 `Batch Status`가 **[Failed]** 인 상태로 종료됨
- 그러나 기본적으로 `Application Exitcode` 는 0 이기 때문에 **Jenkins** 상에서는 **[Success]** 한 빌드로 표시됨



#### Solution

~~~kotlin
@EnableBatchProcessing      // Enable Batch
@SpringBootApplication
class BatchApplication

fun main(args: Array<String>) {
  	/// Get Context of BatchApplication and exitProcess with it
    val exitCode = SpringApplication.exit(runApplication<BatchApplication>(*args))
    exitProcess(exitCode)
}
~~~

- 배치 어플리케이션의 main 함수에서 `runApplication<BatchApplication>(*args)` 의 Return 값인 *ConfigurableApplicationContext* 를 반환받아 이를 이용해 **프로세스의 ExitCode** 로서 종료시 반환
- `Batch Status` 가 프로세스의 `exitCode` 로 전달되기 때문에 배치가 정상적으로 수행되지 않았을 시 빌드 주체자인 **Jenkins** 에도 전달되어 정상적으로 오류를 표시할 수 있음

