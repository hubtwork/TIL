### MariaDB Replication

>✔ DB : MariaDB 10.1
>
>✔ Server : CentOS 7.3



#### Background

- 기존 DB 서버에 스프링 배치를 통한 대규모 데이터 Read / Write 가 진행됨
- 실제 비즈니스로직을 담당하는 서비스가 해당 DB 에 대해 처리를 위해 Read 를 진행할 때 DB 서버의 부하 및 트랜잭션 락이 우려됨
- 이를 위해 DB 의 <u>부하 분산</u>을 위해 **Replication** 을 통해 Master-Slave 구조로 변경하고 **Slave DB 를 Read-Only** 하게 활용



#### Replication

> Replication 에는 일반적으로 두가지 종류가 있다.
>
> Master-Slave 구조 ( 단방향 이중화 ) : **백업 DB**, 부하 분산 **( Master : write , Slave : read )**
>
> Master-Master 구조 ( 양방향 이중화 ) : 부하 분산 및 **데이터 공유** ( Master_1 : read/write **[-share-]** Master_2 : read/write )



#### Master - Slave Replication

- Master : 지정된 데이터베이스에 대해 **Insert / Update / Delete** 등이 발생했을 때 **바이너리 로그**를 생성 후 Slave 로 전달
- Slave : Master 로부터 **바이너리 로그** 를 전달받아 데이터베이스에 반영
- **<span style="color: blue">용도 : DB 백업, 부하 분산</span>**

> <span style="color: red"> **주의** </span>
>
> - **[ 일관성 ]** Master , Slave Connect 전에 DB 상태가 같아야함 ( Dump 이용 ). 
>
> - **[ 호환성 ]** Master , Slave DB 버전은 동일하거나 Slave 가 더 최신이어야 함. 
> - **[ 구동 순서 보장 ]** Master 가 Slave 보다 먼저 구동되어야 함 ( Slave 에서 Master 에 Connection 을 요청하기 때문 ) 

##### Master DB Server Spec

~~~
OS : CentOS 7.3
DB : MariaDB 10.1

Replication DB : matcher	// 복제할 DB
Replication-Ignore DB : batch	// 복제하지 말아야할 DB
~~~

##### Master DB Config

- Master DB 에는 Replication 용 User 가 있어야 함 **( 보안상 root 이용은 지양할 것 )**
- Replication 할 DB 에 대해 권한 부여

~~~mariadb
MariaDB[None] > create user repl@'%' identified by '1234';
MariaDB[None] > grant all privileges on matcher.* to repl@'%' identified by '1234';
MariaDB[None] > flush privileges;
~~~

- **my.cnf** 에 Replication 설정 ( 일반적으로 CentOS 는 "/etc/my.cnf" )

~~~shell
$ vim /etc/my.cnf
[mysqld]
...
## Replication related settings
log_bin_trust_function_creators = 1	# 프로시져 , 트리거 바이너리 로깅 ON
log_bin                         = mysql-bin	# LogBin 설정
server-id                       = 1	# 서버 ID 
max_binlog_size                 = 512M	# 바이너리로그 최대 크기
...

$ service mysql restart	# 설정 완료 후 재시작
~~~

- MariaDB 콘솔 내에서 **Master status** 조회. File 및 Position 확인 ( Slave 에 입력해야함 )

~~~mariadb
MariaDB[None] > show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |    46862 |              |                  |
+------------------+----------+--------------+------------------+
~~~

##### Slave DB Server Spec

~~~
OS : CentOS 7.3
DB : MariaDB 10.1
~~~

##### Slave DB Config

- Replication DB 용 User 생성

~~~mariadb
MariaDB[None] > create user repl@'%' identified by '1234';
MariaDB[None] > grant all privileges on matcher.* to repl@'%' identified by '1234';
MariaDB[None] > flush privileges;
~~~

- **my.cnf** 에 Replication 설정 ( 일반적으로 CentOS 는 "/etc/my.cnf" )

~~~shell
$ vim /etc/my.cnf
[mysqld]
...
# Replication - slave setting
server-id               = 2	# Master 와 서버 아이디를 다르게 해 구분 
log_bin                 = mysql-bin	# LogBin 설정
log_slave_updates	# 관련 바이너리 로그를 가질 수 있도록 on
read_only               = 1	# Slave DB 는 Read-Only 로 쓸 것이라고 명시

# replicate-do-db : replication 할 데이터베이스 
# replicate-ignore-db : replication 하지 않을 데이터베이스
replicate-do-db         = matcher
replicate-ignore-db     = test
replicate-ignore-db     = information_schema
replicate-ignore-db     = mysql
replicate-ignore-db     = performance_schema
replicate-ignore-db     = batch
...

$ service mysql restart 	# 설정 완료 후 재시작
~~~

- Master DB 의 로그 File / position 및 유저를 활용해 DB 의 Master 설정 및 연결 확인

~~~mariadb
MariaDB[None] > CHANGE MASTER TO MASTER_HOST='마스터DB IP',MASTER_PORT=3306 , MASTER_USER='repl',MASTER_PASSWORD='1234', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=46862;
MariaDB[None] > start slave;
MariaDB[None] > show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event	# Connecting to Master 는 미연결 상태
                  Master_Host: 101.101.217.228
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 46862
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 857
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes 	# Yes 가 아닌 Connecting 일 시, 아래 Error 확인 후 개선
            Slave_SQL_Running: Yes
              Replicate_Do_DB: matcher
          Replicate_Ignore_DB: test,information_schema,mysql,performance_schema,batch
          ...
~~~

