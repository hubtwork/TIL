### What is Session ?



#### Session 이란 무엇인가 ?

- `Session` - **서버** 에서 브라우저의 상태를 일정하게 유지시키기 위해 저장하는 데이터
  - 세션은 **사용자(브라우저) 의 연결** 단위로 저장되며 연결 종료시 삭제됨
  - **주의 )** 쿠키와 달리 세션은 로그인 상태와 로그아웃 상태의 세션이 다름 ( 쿠키는 로그인 상태와 상관없이 데이터가 브라우저 내에 저장됨 )

- **세션의 목적**

  - 쿠키의 경우 개인의 웹 브라우저에 저장되기 때문에 유출 / 조작이 쉬워 서버상에 저장하기 위한 방식을 고안

  - **고유성 부여** - 서버 측에서 각 클라이언트에게 고유한 세션아이디를 부여함으로서 클라이언트를 구분할 수 있음

    > 각 클라이언트에 세션아이디를 부여해 페이지 로드시 세션아이디를 이용해 클라이언트마다 필요한 정보를 로드할 수 있음

  - **보안 향상** - 사용자가 사이트를 이용하는 데에 필요한 개인정보를 서버에서 관리하므로 쿠키보다 우수한 보안을 제공

    > 아이디, 비밀번호 등 누출될 시 큰 위험을 초래할 수 있는 민감정보를 서버 측에서 관리함으로 탈취 위험도가 쿠키보다 적음

  - **부하 가능성** - 각 사용자들에 대한 세션들을 서버의 메모리에 저장하기 때문에 서버의 부하를 초래할 수 있음 

    > 서버의 메모리상에 현재 유지중인 세션 정보를 모두 저장하기 때문에 서버의 부하를 초래해 성능을 저하시킬 수 있음



#### Session 은 어떻게 관리하는가 ?

- `session` 은 서버가 접속 중인 클라이언트를 식별하기 위한 수단

- **세션의 동작원리**

  - 클라이언트가 서버에 요청할 때 서버는 헤더의 쿠키 중 `session-id` 가 있는지 확인

    > 서버의 목적 및 형태에 따라 반드시 세션을 사용하지 않을 수 있다 ( ex_ Rest API )
    >
    > 혹은 로그인하지 않은 클라이언트가 상태유지가 필요없는 경우 로그인한 유저에게만 세션을 발급할 수도 있다

  - 만약 `session-id` 가 없다면 서버는 **세션을 생성** 하고 이에 해당하는 `session-id` 를 `SetCookie` 헤더를 통해 전달

    > **알아두기 )** `session` 은 접속이 종료되면 삭제되기 때문에 `session-id` 쿠키는 **Session Cookie** 의 형태이다

  - 만약 `session-id` 가 있다면 서버는 **세션** 을 통해 해당 클라이언트의 **상태유지** 를 지원

    > **Http 통신** 의 무상태성이 가지는 특성을 보완하기 위해 **쿠키** 와 **세션** 을 사용한 것이다

- **세션 활용하기 ( javax.servlet.http.HttpSession )**

  - 세션 생성 및 획득

    - `getSession() `메소드를 이용해 서버에 현재 생성되어있는 세션 정보를 가져올 수 있다
    - 파라미터 *( default = true )* 에 따라서 해당 세션이 존재하지 않을 경우 새 세션을 생성해 반환할지, null 을 반환할지 결정한다

    ~~~java
    HttpSession session = request.getSession();	// 세션이 없을 경우 새로운 세션 생성 및 반환 
    HttpSession session = request.getSession(false);	// 세션이 없을 경우 null 반환
    ~~~

    - `invalidate()` 메소드를 이용해 세션 내의 정보를 초기화할 수 있다

    ~~~java
    session.invalidate()	// 세션 내 정보 초기화
    ~~~

  - 세션에 값 저장 / 조회 / 삭제

    - `setAttribute(String name, Object value)` 메소드를 이용해 세션에 원하는 객체를 저장할 수 있다
    - 만약 이미 존재하는 `name` 이 있을 경우 값을 덮어쓰는데, `value` 가 null 이라면 해당 값을 삭제한다

    ~~~
    session.setAttribute("userInfo", user)
    ~~~

    - `getAttribute(String name)` 메소드를 이용해 세션의 원하는 객체를 조회할 수 있다
    - 만약 주어진 `name` 에 대한 `value` 가 없을 경우 null 을 반환한다

    ~~~java
    UserData user = session.getAttribute("userInfo")
    ~~~

    - `getAttributeNames()` 메소드를 이용해 해당 세션 내에 저장된 모든 `name` 을 조회할 수 있다

    ~~~java
    Enumeration sNames = session.getAttributeNames()
    ~~~

    - `removeAttribute(String name)` 메소드를 이용해 세션의 원하는 객체를 삭제할 수 있다

    ~~~java
    session.removeAttribute("userInfo")
    ~~~

  - 세션 타임아웃 설정

    - `setMaxInactiveInterval(int interval)` 을 이용해 세션의 타임아웃을 설정할 수 있다 ( 초 단위 )

    ~~~java
    session.setMaxInactiveInterval(60*60*4); // 세션의 타임아웃을 60*60*4초 = 4시간으로 설정 
    ~~~

    



### Reference

- https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/http/HttpSession.java
- https://ko.wikipedia.org/wiki/%EC%84%B8%EC%85%98_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)