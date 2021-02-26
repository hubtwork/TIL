### Bash Shell Execute

>✔ **OS :  macOS Big Sur 11.1**



#### Background

- Linux 환경의 서버에서 작업할 소요가 많아짐에 따라 단순한 작업의 경우 쉘 스크립트를 통한 빠르고 간단한 구현의 필요성이 존재함
- 단순하게 명령행을 작성하여 특정 명령을 실행시키는 기본적인 쉘 부터 여러 스크립트 기능들 또한 익히면 도움이 됨



#### Bash Shell Execute

- 스크립트 실행 환경을 **bash shell** 로 지정 ( #! ) 이용
  - /bin/sh - 시스템 지정 기본 shell 사용 ( **현재 필자는 터미널 커스텀을 위해 zsh 을 사용하고 있음** )
  - /bin/bash - bash shell 사용
- <span style="color:red">**[ 주의 ]**</span> 우분투 최신 버전 ( 6.06 ~ ) 의 경우 /bin/sh 의 링크가 **dash shell** 을 가리키고 있음
  - dash shell 과 bash shell 의 키워드 및 문법이 다르기 때문에 주의할 것

~~~shell
# 사용하는 시스템의 기본 shell 이 bash 로 설정되어있는 경우
#!/bin/sh	
# bash shell 을 명시적으로 사용
#!/bin/bash
~~~

- **sh** 혹은 **bash** 를 이용해 쉘 스크립트 ( .sh ) 를 실행

~~~bash
# test.sh 쉘 스크립트 실행
$ bash test.sh
$ sh test.sh
~~~



