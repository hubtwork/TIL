### What is Interrupt ?



#### Interrupt

- OS 는 CPU / RAM 뿐만 아니라 컴퓨터에 연결된 다양한 I/O Device 들을 관리하는 데 이 장치들은 동기적으로 맞물려 작동함

- 동기적으로 맞물리기 떄문에 다른 장치에서 작업이 진행중이라면 CPU 는 그 작업이 끝날때 까지 아무것도 안하면서 기다리는 **불필요한 유휴상태** 에 돌입함

- **Interrupt** - CPU 의 정상적인 프로그램 실행을 **방해** 하여 **현재 실행중인 작업을 멈추고 다른 작업을 우선적으로 진행한 후 다시 복귀** 하게 한다

  - **외부 Interrupt** - 연결된 주변장치들, 특히 I/O 장치들에 의해 발생
    - *Power* - 전원에 문제가 생기는 경우 ( 정전, 파워 누락 등 )
    - *Machine Check* - CPU 가 기능적으로 오류가 생겼을 경우
    - *I/O* - 입출력 장치에 입력이 필요하거나 데이터의 출력이 종료되어 다음 스텝으로 진행해야할 경우
    - *External* - 타이머 / 외부 장치 / 강제 인터럽트를 송신한 경우 ( <u>I/O 디바이스와 외부장치는 다름</u> )
  - **내부 Interrupt** - 프로그램 검사 인터럽트라고도 하며 잘못된 명령, 데이터를 사용할 때 발생
    - *Hardware Interrupt* - 하드웨어의 고장 / 데이터 전송 오류 등 
    - *Operation Failure* - 수행할 수 없는 Division by zero 와 같은 연산이 수행되어 명령어 수행에 실패할 경우
    - *Authority Violation* - 사용자가 접근 불가능한 리소스에 접근을 시도할 경우
    - *Unavailable Operation* - 명령어의 비트 패턴이 정의되지 않았거나 오류가 있을 경우
  - **소프트웨어 Interrupt** - 소프트웨어 실행 중 다른 소프트웨어 실행에 의해 발생
    - *SVC ( SuperVisor Call )* - 사용자가 프로그램을 실행시키거나 Supervisor 를 호출, 복잡한 입출력 처리 등을 수행할 때 **프로그램은 SVC 를 호출** 해 인터럽트를 발생시킴
    - **Time Sharing** 을 위해 인터럽트를 발생시킨 프로그램에 리소스를 할당하는 등의 작업을 수행

  > 소프트웨어 Interrupt 를 Trap 이라고도 한다.
  >
  > *인터럽트* 의 경우 일정한 시점에 발생하지 않기 때문에 **비동기적**
  >
  > *트랩* 의 경우 프로그램 내에서 일정한 시점에 발생하기 때문에 **동기적**



#### Interrupt WorkFlow

- 인터럽트가 발생했을 때 CPU는 원래 진행중이던 **작업을 중단하고 인터럽트에 의한 작업을 수행 후 다시 복귀해 재개** 한다
- **주요 Flow )** 요청 - 중단 - 보관 - 처리 - 재개
- **Work Flow**
  - CPU 에 인터럽트 요청
  - 현재 실행중이던 *Micro Operation* 까지만 수행하고 프로그램 중단
  - 프로그램 상태 보존 - **PC ( Program Counter )** 에 이어서 실행할 명령어를 저장하고 **PCB ( Process Control Block )** 에 프로세스의 현 상태 정보 저장
  - 인터럽트 처리 - **Interrupt Vector** 에서 **ISR ( Interrupt Service Routine )** 에 대한 정보를 탐색
  - 인터럽트 수행 - 파악된 **ISR** 을 통해 인터럽트를 수행한다. 만일 처리중 우선순위가 높은 다른 인터럽트 발생시 해당 인터럽트를 먼저 수행하고 다시 수행하는 방식을 차용
  - 상태 복구 - **RETI ( Return from Interrupt )** 명령어가 수행되면 스택해 저장했던 프로세스 상태를 복구 ( 레지스터 복원 )
  - 프로그램 재실행 - **PC** 에 저장된 명령어 위치를 이용해 프로세스를 재개

> **Interrupt Handler / Interrupt Service Routine**
>
> - 인터럽트 요청에 따른 특정 기능을 처리하는 기계어 코드 루틴
> - 인터럽트 핸들러는 커널에 존재하며 OS 가 요구하는 일을 처리하는 기능적 코드 집합
> - **IF ( Interrupt Flag)** 를 1로 설정해 다른 인터럽트의 발생을 방지할 수도 있음
>
> **Interrupt Vector**
>
> - 인터럽트가 발생할 시 수행해야할 **ISR** 이 어떤 메모리 위치에 적재되어있는지 파악해야함
> - 이를 위해 다양한 **ISR** 의 시작주소를 보관하는 **Interrupt Vector Table** 을 이용해 찾음
>
> **Program Counter**
>
> - 다음에 실행될 Micro Operation 의 주소를 갖고 있는 레지스터
> - 명령어 포인터라고도 불리며 인터럽트 발생시 프로세스를 재개하기 위한 보관소의 역할 또한 수행함



#### Interrupt Priority

- 인터럽트를 수행 중 다른 인터럽트가 발생할 경우 이를 처리하기 위한 우선순위가 존재함

- ~~~
  Power > Machine Check > External > I/O > Program Check > SVC
  ~~~

- **우선순위 판별 전략** - 인터럽트간 우선순위를 판별하기 위한 전략

  - **Polling** - 우선순위가 높은 인터럽트부터 요청플래그를 검사해 *ISR* 을 수행하는 <u>소프트웨어적인 방식</u> 
    - 인터럽트에 대한 Scan Time 이 늘어 응답시간이 지연되지만 하드웨어 추가에 대한 소요가 감소
  - **Daisy Chain** - 인터럽트가 발생하는 모든 장치를 우선순위에 따라 직렬로 연결하는 <u>하드웨어적인 방식</u>
  - **Parallel Priority** - 인터럽트가 발생하는 모든 장치를 직렬로 연결하는 <u>하드웨어적인 방식</u>
    - Mask Register 에 장치별 우선순위를 판별하기 위한 bit 를 설정하고 인터럽트 수행시 우선순위가 낮은 bit 의 장치들을 비활성화하고 우선순위가 더 높은 bit 의 장치들만 인식 가능하게 하여 우선순위 기반 인터럽트 처리를 가능하게 함





### REFERENCE

- https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%9F%BD%ED%8A%B8_%ED%95%B8%EB%93%A4%EB%9F%AC
- https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%9F%BD%ED%8A%B8
- https://hibee.tistory.com/264