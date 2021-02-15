### Jenkins 설정



#### JDK 설정

- Manage Jenkins > Global Tool Configuration
- JDK > JDK Installations > ADD JDK
- **Install automatically** 체크박스 해제
- **JAVA_HOME** 탭에 **젠킨스 서버의 JDK 설치 디렉토리** 기입

~~~shell
# JAVA_HOME 환경변수 확인을 통해 현재 JDK 설치 디렉토리 확인
$ echo $JAVA_HOME
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64
~~~



#### Git 설정

- Manage Jenkins > Global Tool Configuration
- Git > Git Installations > ADD Git
- **Install automatically** 체크박스 해제
- **Path to Git executable** 탭에 **젠킨스 서버의 Git 설치 디렉토리** 기입

~~~shell
# which 커맨드를 이용해 Git 설치 디렉토리 확인
$ which git
/usr/local/bin/git
~~~



#### Gradle 설정

- Manage Jenkins > Global Tool Configuration
- Gradle > Gradle Installations > ADD Gradle
- **Install automatically** 체크박스 해제
- **GRADLE_HOME** 탭에 **젠킨스 서버의 Gradle 설치 디렉토리** 기입

~~~shell
/usr/lib/gradle
~~~

