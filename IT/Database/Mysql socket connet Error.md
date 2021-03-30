

### Mysql / MariaDB - Socket connet Error



#### Problem

~~~shell
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'
~~~

- Linux / OSX 환경에서 시스템을 재시작하거나 어떤 특정 시점에서 DB 서비스 시작을 시도할 때 `Socket Error`  에 의해 서비스 이용 불가 상황 발생



#### Analyze

- **원인 분석**

  **case 1** - DB 서비스 구동을 위해 `mysql.sock` 소켓 정보를 참조해야 하는데 `my.cnf` 내에서 소켓 정보가 정의되어 있지 않은 경우

  **case 2** - 특정 유저로 서비스를 실행할 때 `mysql.sock` 이 저장된 디렉토리인 `/var/lib/mysql` 의 권한이 허용되지 않은 경우

  **case 3** - `mysql.sock` 의 심볼릭링크가 삭제되었거나 다른 경로에 대해 설정되어 있는 경우

  **case 4** - DB 서비스가 실행중이지 않거나 비정상적인 프로세스로 작동중인 경우



#### Solution

- **case 1** - `my.cnf` 의 socket 설정을 확인 및 수정

  ~~~shell
  $ vi /etc/my.cnf
  ---
  [server]
  socket	=	/var/lib/mysql/mysql.sock
  ...
  [client]
  socket	=	/var/lib/mysql/mysql.sock
  ~~~

- **case 2** - `/var/lib/mysql` 디렉토리에 대해 특정 유저 접근 권한 허용

  ~~~shell
  # mysql 서버 종료
  $ systemctl stop mysqld
  # 디렉토리 권한 변경
  $ chmod -755 -R /var/lib/mysql/
  # 원하는 유저명을 기입해 해당 유저의 소유권 변경
  $ chown -R mysql:"userName" /var/lib/mysql/
  # mysql 서버 재시작
  $ systemctl start mysqld
  ~~~

- **case 3** - `mysql.sock` 심볼릭 링크 재설정

  ~~~shell
  # 소프트 링크를 통해 시스템 라이프사이클 동안 심볼릭 링크 지정
  $ ln -sf /tmp/mysql.sock /var/lib/mysql/mysql.sock
  ~~~

- **case 4** - DB 서비스의 프로세스를 모두 종료 후 재실행

  ~~~shell
  # mysql 프로세스 확인, 종료 후 재시작
  $ ps -A | grep mysql
  $ sudo pkill mysql
  $ systemctl start mysqld
  ~~~

