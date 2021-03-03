### MariaDB Memory Relevant Setting

>✔ DB : MariaDB 10.1
>
>✔ Server : CentOS 7.3 **[ Memory 16GB ]**



#### Background

- 서비스를 위해 별도의 **MariaDB 서버** 를 운용중에 서버의 스펙을 효율적으로 사용하고 있는지 체크
- DB 서버의 '스펙 요소' 는 **메모리** 가 가장 큰 영역을 차지함
- 이에 **MariaDB** 의 메모리 영역과 메모리 적정 설정에 대해 체크



#### Memory in MariaDB

- MariaDB 의 메모리는 **Global / Session** 으로 나눌 수 있음
- **Global Memory**
  - **Important >** MariaDB 는 서버를 시작한 후 
    - 1) 서버 운영에 필요한 최소 메모리만 사용하다가
    - 2) 메모리의 허용치까지 사용량이 증가함. 
    - 허용치까지 메모리가 도달할 시 <u>메모리 반환 없이</u> 허용치 내에서 **메모리를 활용해 운영**함
  -  **Difference >** OracleDB 의 경우 서버 시작과 동시에 **허용치만큼 메모리를 할당받아 운영**
  - 클라이언트의 스레드 수와 무관하게 하나의 메모리 영역을 할당함. 필요에 의해 N개의 메모리를 할당받았더라도 이는 모든 스레드에 의해 공유됨.
  - **Innodb_buffer_pool_size** : InnoDB 엔진이 디스크에서 데이터를 메모리에 캐싱, 데이터 변동사항을 버퍼링함. <span style="color: red">추천 설정 ) </span> 전체 메모리의 **60~80%**
  - **innodb_log_buffer_size** : InnoDB 엔진의 로그를 저장할 때 쓰는 버퍼의 크기.
  - **Key_buffer_size** : InnoDB 엔진이 아닌 **MyISAM** 엔진 사용시 인덱스를 저장하는 키 버퍼의 크기. 
    - **주의 )** 인덱스를 캐싱하는데 사용되기 때문에 크기가 클 필요가 없음
  - **tmp_table_size** : 메모리에 생성되는 임시 테이블의 크기

- **Session Memory**
  - **Important > **원칙적으로는 동시에 접속중인 세션 수에 대한 측정이 필요하나 이는 변칙적이기 때문에 **Max Connection** 의 메모리 사용을 고려해 측정함
  - 클라이언트의 스레드가 쿼리를 처리할 때 사용되는 메모리 영역. 클라이언트 스레드 별로 할당되며 각 스레드 간의 공유는 절대로 일어나지 않음.
  - 스레드가 별도의 쿼리를 진행중이지 않을 때는 기본적인 영역만 할당되며 **JOIN / SORT** 등 필요할 때에는 설정값만큼 추가적으로 메모리를 할당.
  - **join_buffer_size** : 주어진 조인 조건이 부적절해 드리븐 테이블의 검색이 풀 테이블 스캔으로 유도될 경우 사용됨.  
    - **주의 )** 모든 조인에 대해 할당되는 것이 아님
  - **sort_buffer_size** : 실해할 소트에 대해 **적절한 인덱스가 없는 경우** 할당되는 메모리 버퍼의 크기. 
    - **주의 )** 해당 버퍼의 크기가 *너무 클 경우* 클라이언트 스레드에 의한 메모리 누수가 많아지며 *너무 작을 경우* 디스크의 직접적 사용률이 높아짐. 적절한 값에 대한 측정이 필요.
  - **read_buffer_size** : **풀 테이블 스캔**이 발생할 경우에 사용되는 버퍼의 크기.
  - **read_rnd_buffer_size** : 인덱스를 사용할 수 없는 소트에 대해 **소팅 결과를 메모리에 저장** 해 불필요한 디스크 접근을 없앨 때 사용되는 버퍼의 크기.
  - **thread_stack** : 각 클라이언트 스레드에 할당되는 스택의 사이즈. 
    - **주의 )** 복잡한 SQL 쿼리의 수행에 필수적이나, 너무 클 경우 메모리 누수 발생.
  - **binlog_cache_size** : 각 트랜잭션이 시작될때 미리 확보해 바이너리 로그를 저장할 때 사용되는 메모리의 크기.
  - **Max connection** : 최대 동시 커넥션 수.



#### Memory Setting Recommendation

- **my.cnf** 내의 각 변수를 이용해 설정
- MariaDB 서버는 기본적으로 **(350 ~ ) MB** 를 이용하며 performance_schema 가 **(150 ~ ) MB** 의 메모리를 할당받음
- **OS - File Buffering** 을 위해 **메모리의 10% 전후** 가 할당됨
- 이에 아래 두 메모리 영역의 합을 추정 및 재설정해 메모리 성능을 극대화
- **Global Memory** ( Innodb_buffer_pool_size + Key_buffer_size + innodb_log_buffer_size + tmp_table_size )
- **Session Memory** ( (join_buffer_size + sort_buffer_size + read_buffer_size + read_rnd_buffer_size + thread_stack + binlog_cache_size) x Max connection )