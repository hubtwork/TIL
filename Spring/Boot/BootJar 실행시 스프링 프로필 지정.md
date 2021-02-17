### Multi-Module 프로젝트 In Gradle

>✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
>✔ **OS :  macOS Big Sur 11.1**
>
>✔ **Lang: Kotlin**



- Gradle > 빌드를 원하는 모듈 > Tasks > Build > BootJar
- <span style="color: red"> **예제**</span> Spring Boot Batch 실행 

~~~bash
$ java -jar ./"project Directory"/build/"projectName + version".jar \	# Spring BootJar Execute
--spring.profiles.active="Spring Profile" \	# Spring Boot Profile
--job.name=loadMatches \	# Spring Batch Job Name
buildNumber=1 	# Unique Job Parameter
~~~



