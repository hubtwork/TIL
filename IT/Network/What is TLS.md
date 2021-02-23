### What Is TLS ? 



#### TLS의 전신, SSL

- **<span style="color:red">S</span>ecure <span style="color:red">S</span>ocket <span style="color:red">L</span>ayer** - Netscape 사에서 만든 인터넷 보안 프로토콜 ( ~ ver 3.0 )
- **CA** ( Certificate Authority ) 에 의해 서버와 클라이언트 사이의 인증과정을 진행
- SSLv2 - IETF 에 의해 2011년 **<span style="color:red">Abort</span>**
- SSLv3 - IETF 에 의해 2015년 **<span style="color:red">Abort</span>**
- TLS 1.0 제안

> **SSL** 을 개선해 **TLS** 를 만들었으나 둘은 호환되지 않음 !
>
> **TLS** 를  **SSL** 이나 **TLS/SSL** 로 말하기도 함 !



#### TLS란 무엇인가

- **<span style="color:red">T</span>ransport <span style="color:red">L</span>ayer <span style="color:red">S</span>ecurity** - IETF 에 의해 SSLv3 를 개선해 표준으로 지정됨 [ RFC 8446 ]
- 패킷을 암호화해 통신 당사자가 신뢰할 수 있는지 확인 및 전송 데이터의 도청 / 변조를 방지
- **대칭키 , 비대칭키 방식을 혼용해서 사용**
  - 비대칭키 방식이 보안적 측면에서 뛰어나지만 계산을 위해서는 CPU 리소스가 많이 소모됨
  - Handshake 페이즈에서 **비대칭키** 를 이용해 대칭키를 생성 및 공유
  - Communication 페이즈에서 이전에 만들어진 **대칭키** 를 이용해 데이터를 암호화 및 전송
- TLS 가 추가되면 일반 통신에 비해 보안 강화 및 성능 저하 > **HTTP/2** 를 통해 해결



#### TLS Handshake Protocol

- **Key Establishment**
  - [ 클라이언트 ] 랜덤 데이터(Client), 사용가능한 암호화 방식과 지원되는 TLS 버전 전송
  - [ 서버 ] 랜덤 데이터(Server), 채택한 암호화 방식과 채택된 TLS 버전, 인증서 전송
  - [ 클라이언트 ] 인증서가 Trusted Root CA 에 의해 발급되었는지 확인한 후 해당 CA 의 공개키를 이용해 복호화
  - [ 클라이언트 ] 랜덤 데이터(Client) + 랜덤 데이터(Server) 를 조합해 pre-master-secret 생성
  - [ 클라이언트 ] pre-master-secret 을 인증서의 public key 를 이용해 암호화 및 전송
  - [ 서버 ] 서버의 private key 를 이용해 pre-master-secret 을 복호화 후 Session Key 생성
- **Key Derivation**
  - pre-master-secret 을 **48bit 의 master-secret** 으로 변환 ( 채택된 암호화 방식에 따라 pre-master-secret 은 다를 수 있기 때문 )
  - 해시함수를 이용해 아래 4개 Session Key 유도 ( AEAD 암호화의 경우 6개 Key )
    - 클라이언트 MAC Key - 서버가 클라이언트로부터 받은 데이터의 무결성 확인
    - 서버 MAC Key - 클라이언트가 서버로부터 받은 데이터의 무결성 확인
    - 클라이언트 Encription Key - 서버가 클라이언트로부터 받은 데이터 암호화
    - 서버 Encription Key - 클라이언트가 서버로부터 받은 데이터 암호화
    - ( AEAD 암호화 이용시 ) 클라이언트 / 서버의 IV Key - 각각 서버, 클라이언트에서 AEAD 암호로 사용
  - 해당 Key 들은 전송 계층을 통해 전송되는 레코드, 전송된 메세지 등을 보호함

- **Encryption Notification**
  - **ChangeCipherSpec** 메세지를 전송해 이전 과정에서 만든 Session Key 를 이용해 통신할 것을 통지
  - **Finished** 메세지를 전송해 각자의 TLS Handshake가 끝났음을 통지
  - TLS 프로토콜에 의해 보호되는 통신이 구성됨



### REFERENCE

- https://www.comparitech.com/blog/information-security/tls-encryption/