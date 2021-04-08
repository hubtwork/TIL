### What is Cookie?



#### Cookie 란 무엇인가 ?

- `Cookie` - **웹 브라우저(클라이언트)** 에 저장되는 4kb 이하의 `key - value` 형태의 임시 데이터

  - **웹 브라우저** 마다 저장되는 쿠키는 다름 ( ex_ Safari 의 쿠키는 크롬에서 사용할 수 없음 )
  - **Why ?** 서버 입장에서 웹 브라우저가 다를 경우 다른 사용자로 인지하기 때문

- **쿠키의 목적**

  - 페이지 이동 시 사용자 정보 등 필요한 정보를 서버에 전송 ( 쿠키가 없으면 페이지 이동시 서버가 직접 필요한 데이터를 파라미터로 넘겨야 함 )

  - **세션 관리** - 아이디, 로그인 상태, 장바구니 등 서버가 페이지를 로드할 때 알아야할 정보를 저장

    > 자동 로그인 허용과 함께 로그인이 성공했을 시 쿠키에 자동로그인 상태를 아이디와 저장해 재접속시 자동 로그인될 수 있도록 함

  - **트래킹** - 최근 각 사용자의 행동 패턴 분석을 통한 개인화가 메인 이슈인데 이를 위한 행동 패턴을 분석해 저장

    > 최근 검색 기록등을 저장해 이에 맞는 광고를 사용자에게 제공

  - **데이터 보관** - 몇 일간 다시 보지 않기 등 사이트 이용에 대한 개인의 설정 데이터를 저장

    > 쿠키에 날짜를 기록해두고 사이트 접속시의 시간과 비교해 특정 공지사항 다이얼로그를 띄우지 않도록 할 수 있음



#### Cookie 의 구성요소와 종류

- **쿠키의 구성요소**

  - `Name` - 쿠키의 이름. 쿠키에서 정보를 인식할 Key 값
  - `Value` - 쿠키의 값. 쿠키에서 Key 를 통해 가져올 Value 값

  ---

  - `Expires / Max-Age` - 쿠키의 만료시간. `Set-Cookie` 에 해당 필드가 포함될 경우 **Permanant Cookie** 로 간주함

    > `Expires=Thu, 8 Apr 2021 07:28:00 GMT;` - 해당 시점에 쿠키 만료
    >
    > `Max-Age=130000;` - 130000 초 후 쿠키 만료
    >
    > 둘 다 설정되어있을 경우, `Max-Age` 를 우선시해 만료기일을 결정하며 클라이언트의 상대적 시점으로 지정

  - `Domain` - 쿠키가 적용될 도메인 정보

    > `Domain=hubtwork.com;` - 해당 쿠키가 `hubtwork.com` 호스트에서 적용됨을 명시
    >
    > 만약 설정되지 않았을 경우, 현재 사이트 URI 를 기준으로 적용

  - `Path` - 쿠키가 사용될 경로 정보

    > `Path=/product/cart/;` - 해당 쿠키가 지정 `hubtwork.com/product/cart/` (도메인+경로) 에서 적용됨을 명시
    >
    > 지정된 리소스 URL 의 하위 디렉토리 또한 포함됨

  - `Secure` - 쿠키의 보안 적용 여부

    > `Secure;` - 해당 쿠키가 보안이 보증되는 `https` 통신에서만 적용 가능함을 명시

  - `HttpOnly` - 쿠키의 조회 제한 여부

    > `HttpOnly` - 해당 쿠키를 `document.cookie` 를 이용해 쿠키에 접근하는 것을 방지함

- **쿠키의 종류**

  - **Session Cookie** - 클라이언트가 종료될 때 삭제되는 쿠키 ( `Expires / Max-Age` 가 설정되지 않은 경우 )
  - **Permanent Cookie** - 클라이언트가 종료되어도 만료될 때 까지 유지되는 쿠키 ( `Expires / Max-Age` 가 설정된 경우 )
  - **Secure Cookie** - `https` 통신에서만 사용가능한 쿠키 ( `Secure` 가 설정된 경우 )
  - **Third-Party Cookie** - 현재 방문한 도메인이 아닌 다른 도메인의 쿠키 ( 광고배너의 유입 경로 추적 등에 사용됨 )



#### Cookie 는 어떻게 주고받는가 ?

- **쿠키의 동작원리**

  - 클라이언트가 서버에 요청하면 쿠키에 저장할 정보를 `SetCookie` 헤더와 함께 전달

    > 예시 ) 로그인 아이디와 장바구니에 담은 ProductId 와 수량을 함께 쿠키에 저장

    ~~~http
    Set-Cookie: "cookie-Name"="cookie-Value"
    [ example ]
    Set-Cookie: login_id=hubtwork
    Set-Cookie: p_1314=2
    Set-Cookie: p_141=8
    ~~~

  - 클라이언트는 `SetCooke` 에 해당하는 쿠키정보를 웹 브라우저의 메모리 혹은 파일형태로 저장

  - 클라이언트가 이후 서버에 요청할 때 해당 요청에 부합하는 쿠키가 있는지 확인해 쿠키를 포함해 전달

    ~~~http
    Cookie: "cookie-Name"="cookie-Value"
    [ example ]
    Cookie: login_id=hubtwork; p_1314=2; p_141=8
    ~~~

    > 요청 시에 해당 요청에 유효한 쿠키를 전송해 장바구니에 담은 정보를 DB 가 아닌 쿠키로 관리하여 DB 부하를 줄일 수 있음



### REFERENCE

- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie

