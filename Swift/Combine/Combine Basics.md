### Combine Basics

>✔ **Xcode : Version 12.4 (12D4e)**
>
>✔ **OS :  macOS Big Sur 11.3 Beta (20E5210c)**



#### Combine

- `Combine` - 비동기 이벤트 처리를 위한 `SwiftUI` 맞춤형 데이터 바인딩 프레임워크
- **Combine** 의 특징
  - **Combine** 은 <u>새로운, 통합된 트렌디하고 심플한</u> 선언적인 Swift API 이다.
  - **새로운** - `SwiftUI` 와 함께 `RxCocoa + RxSwift` 를 대체하는 새로운 공식 ios 패러다임
  - **통합된** - `Notification Center, Delegate, ... ` 등 기존의 서로 다른 비동기 API 들을 다루기 위한 **통합 비동기 API**
  - **트렌디** - **Rx** 가 아닌 **Reactive Streams** 기반의 선언적 이벤트 처리 지원 ( Spring 의 `Reactor` 처럼 )
  - **심플함** - 기존 Swift 에서 비동기 구현을 위해 `Delegate, Completion Handler, ...` 등의 구현 대신 <u>이벤트 소스에 대한 프로세스 체인</u> 을 구현
- **Combine** 의 구성
  - `Publisher` - 시간이 지남에 따라 변할 수 있는 데이터의 값들을 원하는 친구들 ( Subscriber ) 에게 방출해줌
  - `Operator` - 데이터에 대한 처리 및 연산을 담당하여 여러 Operator 가 맞물려 복잡한 비즈니스 로직을 구현
  - `Subscriber` - 방출되어 전달받은 값을 이용해 어떠한 동작을 처리 및 수행



#### Subscription

- `Subscription` - `Combine` 이 제공하는 데이터에 대한 프로세스 체인
- **Subscription 의 장점**
  - **Subscription** 은 하나의 데이터 구독 플로우를 전부 설명할 수 있기 때문에 각 비즈니스 로직의 표현에 용이
  - **Subscription** 은 각 비동기 이벤트들을 처리하기 위한 체인을 커스텀 코드와 에러핸들링을 이용해 표현할 수 있음
  - 별도의 콜백 구현이 필요없이 시스템에서 **Subscription** 을 관리해줌
  - **Cancellable** 프로토콜을 이용해 비동기 이벤트를 관리하는 데에 필요한 오버헤드 및 부작용들을 배제시킴
- **Subscription Chain**
  - **Publisher** - `output` , `completion` , `failure` 를 이용해 데이터를 방출
    - 하나의 **Publisher** 는 지속적인 `output` 의 방출할 수 있으며 하나의 `completion` 또는 `failure` 를 반환함으로서 작업을 마침
    - `output` - Subscriber 에게 필요한 값을 전달
    - `completion` - Publisher 가 수행한 completion 을 반환 [ 종단점 ]
    - `failure` - completion 에 대한 에러 핸들링을 지원 [ 종단점 ]
  - **Operator** - `upstream` , `downstream` 을 이용해 **Publisher** 들을 이용한 체인을 구축
    - 하나의 **Subscription** 에서 **Operator** 는 다수 존재하여 추가적인 값의 변경을 수행
    - `upstream` - 구독하는 **Publisher** 로부터 값을 전달받아 추가 작업 수행
    - `downstream` - 다음 **Operator** 또는 **Subscriber** 에게 값을 전달
  - **Subscriber** - `sink` , `assign` 을 이용해 최종적으로 값을 전달받음
    - 구독하는 **Publisher** 의 Output Type 을 Input Type 으로 받아 작업 수행
    - `sink` - Closure 를 이용해 output 이나 completion 의 값을 받아 처리
    - `assign` - 전달받은 output 이나 completion 을 바로 정해진 Instance 의 **property** 에 바인딩 ( `failure` 가 `Never` 타입이어야 함 )

<p align="center"><img src="../../assets/img/combine_process.png" alt="Imgur" width="300"/> </p>



### REFERENCE

- https://sujinnaljin.medium.com/combine-combine-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-ac726ac40b07
- https://developer.apple.com/documentation/combine