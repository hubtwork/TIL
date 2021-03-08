### What's new in TLS 1.3 ?



#### TLS 1.0

- 1999년, *IETF* 가 *Netscape* 에게 **SSL** 을 양도받으면서 SSL 3.0 의 업그레이드 버전으로서 공개
- 기존 SSL 버전이 가지고 있던 취약점들에 대해 대폭 수정이 이뤄져 릴리즈
- 하지만 취약점 개선 외엔 기존 SSL 과 큰 차이가 없어 SSL 3.0 과 동일한 버전으로 간주됨



#### TLS 1.1

- 2006년 4월, **CBC 블록 암호 공격에 대한 보안** 과 **IANA 등록 파라미터 지원** 등을 추가해서 공개



#### TLS 1.0 / 1.1 지원 중단

- 해당 두 버전의 경우 **Beast** 와 **Poodle** 공격과 같은 보안 취약점이 아직 존재함
  - **Beast ( Browser Exploit Against SSL/TLS )** - 클라이언트에서 HTTPS 쿠키를 임의로 해독하고 타겟 유저의 세션을 하이재킹 가능하게 함
  - **Poodle ( Padding Oracle On Downgraded Legacy Encryption )** - SSL/TLS 통신에서 클라이언트, 서버의 버전 중 하나라도 최신 버전이 아닐 경우 이전 버전의 프로토콜로 연결되는데 이걸 이용해 공격자가 통신 프로토콜을 SSL 3.0 까지 다운그레이드 시키고 패딩을 이용해 공격함

- 2020년, IETF 는 기존의 TLS 표준에서 오래된 구버전 호환을 파기하는 안을 심사중에 있음
- 코로나 여파에 의해 해당 심사는 지연되고 있으나 브라우저 벤더들에 의해 2020년 **TLS 1.0 / 1.1 지원 중단**



#### TLS 1.2

- 2008년 8월, *IETF* 에 의해 **RFC 5246** 발표
- 보안 강화를 위해 기존의 **SHA1** 해싱 알고리즘을 **SHA 2** 로 변경함
- 커넥션이 열릴 때 TLS 핸드셰이크를 통해 암호화 방식 / 키 를 교환하는 과정이 진행되어 **데이터에 대한 탈취, Injection 등의 위협에 대한 보안이 강화** 되었지만 이를 위해 커넥션 마다 2RTT 가 추가되기 때문에 연결마다 속도 저하가 발생함



#### TLS 1.3

- 2018년 8월 10일, *IETF* 에 의해 **RFC 846** 발표
- TLS 발표 이후 대대적인 첫 개편으로 속도 향상과 보안에 있어 중점을 둔 리빌딩이 이루어짐
- **SNI 필드** 의 암호화 규격까지 적용할 시 통신 상 데이터의 평문 부분이 없기 때문에 검열조차 불가함
- 아직까지 TLS 1.2 가 보편적이며 1.3 의 경우 일부 서버에 의해 사용되고 있음 ( ex_ 구글 , 네이버 등 )
- **개선점**
  - **Unsafe Algorithm Deletion** - 보안 위협을 줄이기 위해 기존의 지원 기능을 대폭 간소화 / 강화함
    - *Key Exchange* - ```RSA, DH, ECDH, SRP, PSK``` 에서 ```ECDHE, PSK, PSK + ECDHE``` 로 변경
    - *Signing* - ```RSA, DSA, ECDSA``` 에서 ```RSA, ECDSA, EdDSA``` 로 변경
  - **1 Handshake** - 기존에 2RTT 에 걸쳐 이루어지던 핸드셰이크를 1RTT 만에 해결할 수 있도록 개선함
    - *Cipher Suite 의 단순화* : 기존에 대칭키 교환 방식, 인증서 서명 알고리즘, 대칭키 암호화 알고리즘, HMAC 알고리즘 **총 4가지 암호화 방식을 묶어 합의하는 방식**을 취했던 것과 달리 <u> 암호화 알고리즘, 키 교환 방식, 인증서 서명 알고리즘</u> 을 각각 합의하게 함으로서 단순화시키고 1RTT 만에 해결할 수 있게 함
    - *키 교환 방식의 단축* : 키 교환 방식으로 ECDHE 와 DH 만을 지원하게 되면서 지원하는 키 교환 방식에 대한 가정이 가능해졌으며 이로 인해 Cipher Suite 를 합의하는 과정을 단축함
  - **0-RTT Resumption** - 연결을 재개할 때, 공유되는 Session ID 값을 전달하며 재연결하던 기존의 방식과 달리 PSK ( Pre-Shared Key ) 를 TCP Handshake 시에 함께 보내 PSK 에 대한 검증을 수행 후 암호화된 데이터를 바로 보낼 수 있게 함으로서 **TLS Resumtion Handshake  을 0-RTT 로 만듦**





### REFERENCE

- https://namu.wiki/w/TLS#s-1.3.4
- https://ko.wikipedia.org/wiki/TLS_%EA%B5%AC%ED%98%84%EC%9D%98_%EB%B9%84%EA%B5%90
- https://www.dailysecu.com/news/articleView.html?idxno=2138
- https://www.trendmicro.com/en_us/research/14/j/poodle-vulnerability-puts-online-transactions-at-risk.html