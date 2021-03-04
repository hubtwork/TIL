### Query Optimization, For Better DB



#### Background

- DB 에 다량의 데이터 삽입 및 처리가 발생하기 시작하면서 **쿼리 최적화의 중요성**이 커지고 있음
- 쿼리의 최적화 여부에 따라 DB 의 성능이 비약적으로 상승하는 다양한 연구결과가 공개됨
- **쿼리를 최적화** 할 수 있는 방법을 알아보고 유의하여 활용해야 함



#### Query Optimization

- 상황에 따라 **필요한 Column** 만을 불러오자

  - 많은 필드를 불러올 수록 DB 가 로드할 때 갖는 부담은 가중되기 때문에 속도저하가 일어난다. 

  - 필요한 칼럼만 로드한다면, 결과 테이블의 크기는 줄어들고 네트워크 트래픽 또한 감소한다.

  - ~~~mariadb
    # Original
    select * from matchList;
    # Improved
    select matchId, season from matchList;
    ~~~

- **Having** 절을 피하자

  - **Having** 절은 모든 열을 선택한 후 필터링하는 데에 사용된다.

  - 즉, 조건절에 따른 결과 테이블을 얻은 후 **Having** 에 맞게 다시 데이터를 필터링하므로 불필요한 연산이 발생한다.

  - 가급적이면 **Where** 절을 이용해 명시적으로 원하는 결과를 로드하도록 한다.

  - ~~~mariadb
    # Original
    select accountId, count(matchId)
    from user_match;
    group by accountId
    having accountId > 1000;
    # Improved
    select accountId, count(matchId)
    from user_match;
    where accountId > 1000;
    group by accountId;
    ~~~

- **Where** 절 사용시, 추가적인 연산을 수행하지 말자

  - DB 내에 저장된 데이터에 별도의 연산 수행해 조건절로 사용하면 모든 데이터에 대해 연산을 수행한 후 조건을 파악한다.

  - 즉, DB 에 대한 **Full Scan** 이 선행되고 나서 불필요한 연산이 발생하기 때문에 성능저하를 일으킨다.

  - 인덱스를 이용한 스캔이 진행될 수 있도록 칼럼에 대한 추가연산은 지양하자.

  - ~~~mariadb
    # Original
    select matchId from matchList
    where round(season/2) = 10;
    # Improved
    select matchId from matchList
    where season between 5 and 14;
    ~~~

- **Distinct** 사용을 피하자

  - **Distinct** 연산을 사용하면 정렬을 선행하고 중복되는 것을 파싱하기 때문에 속도저하가 발생한다.

  - **Primary Key** 가 설정되어 있다면 데이터의 *유일성* 이 보장되므로 **Distinct** 사용을 지양하자.

  - ~~~mariadb
    # Original
    select distinct matchId from matchList;
    # Improved
    select matchId from matchList;
    ~~~

  - 중복값 제거가 필요하다면 **Exists** 를 이용하는 방법이 있다.

  - 서브쿼리와 함께 적절하게 사용한다면 **Exists** 는 Full Scan 없이 중복을 제거하는 방법이 된다.

  - ~~~mariadb
    # Original
    select distinct m.matchId, m.matchData
    from match m, user_match u
    where u.matchId = m.matchId
    # Improved
    select distinct m.matchId, m.matchData
    from match m
    where exists ( select 'X' from user_match u where u.matchId = m.matchId )
    ~~~

- 인덱스 컬럼에 대해서는 **In** 을 이용하자

  - **In** 연산은 인덱스 컬럼에 대해 효과적으로 작동한다.

  - DB 의 **Optimizer** 가 **In** 리스트를 **Index** 의 정렬순서와 일치시켜 효율적으로 검색할 수 있게 한다.

  - ~~~mariadb
    # Original
    select * from matchList
    where matchType like "%Rank"
    # Imporved
    select * from matchList
    where matchType in ("Solo Rank", "Flex Rank")
    ~~~

- **Union** 대신 **Union All** 을 사용하자

  - **Union** 과 **Union All** 의 가장 큰 차이점은 중복 검사의 유무이다.

  - **Union All** 은 중복검사를 하지 않기 때문에 비교했을 때 비약적인 속도 향상을 확인할 수 있다.

  - ~~~mariadb
    # Original
    select matchId from matchList
    union
    select matchId from user_match;
    # Improved
    select matchId from matchList
    union all
    select matchId from user_match;
    ~~~

- **Or** 의 사용을 피하자

  - **Or** 을 사용하여 비교 연산을 진행할 경우 인덱스 스캔을 하지 않고 **Full Scan** 이 발생한다.

  - 이는 쿼리를 수행할 때 심각한 속도저하를 유발하기 때문에 **Union All** 을 이용하여 구현하는 것이 바람직하다.

  - ~~~mariadb
    # Original
    select * from matchList m 
    inner join statistics s
    on m.timeline = s.sTime or m.timeline = s.eTime
    # Improved
    select * from matchList m 
    inner join pipelinedMatch p
    on m.timeline = s.sTime 
    union all
    select * from matchList m 
    inner join pipelinedMatch p
    on m.timeline = s.eTime
    ~~~



### REFERENCE

- https://www.ijstr.org/final-print/oct2015/Query-Optimization-Techniques-Tips-For-Writing-Efficient-And-Faster-Sql-Queries.pdf