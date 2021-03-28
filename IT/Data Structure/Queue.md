### Data Structure - Queue



#### Queue

- `Queue` - **FIFO ( First In First Out )** 형태의 선형 ( Linear ) 자료구조
- **FIFO** - 처음에 들어온 데이터가 가장 먼저 나가는 형태로서 **선입선출** 이라고도 함
  - 예시로 **CPU** 의 `I/O Management` 를 들 수 있다
  - 프린터, 프로세스 등을 관리할 때 입력이 들어온 순서대로 처리할 수 있도록 `Queue` 를 이용해 구현한다
- **Queue Operation**
  - `enQueue` - 큐에 데이터 삽입
  - `deQueue` - 큐의 첫 데이터 삭제
  - `front` - 큐의 첫 데이터 반환
  - `rear` - 큐의 마지막 데이터 반환
  - `size` - 현재 큐 내의 데이터 개수를 반환
  - `isEmpty` - 큐가 비었는지 여부를 반환 ( `size` == 0  *or* `front` == `rear` )

<p align="center"><img src="../../assets/img/queue.png" alt="Imgur" width="400" /></p> 

- **Queue Implementation**

  - **배열을 이용한 구현**

  > 구현이 간편하며 **python** 과 같이 동적 배열을 지원하는 언어의 경우 크기에 제한을 받지 않아 구현에 제약사항이 없음
  >
  > **python** 의 경우 `dequeue()` 를 List 의 `pop(0)` 을 이용해 구현할 수 있지만 그 뒤 인덱스들의 모든 데이터를 한칸씩 당겨오는 연산을 수반하기 때문에 **시간복잡도 O(N)** 이 추가됨

  ~~~python
  class Queue:
      def __init__(self):
          self.queue = []
      
      def enqueue(self, data):
          self.queue.append(data)
      
      def dequeue(self):
          if self.isEmpty():
              return "Error - Queue is Empty"
          return self.queue.pop(0)
    
      def front(self):
          if self.isEmpty():
              return "Error - Queue is Empty"
          return self.queue[0]
  
      def rear(self):
          if self.isEmpty():
              return "Error - Queue is Empty"
          return self.queue[-1]
    
      def isEmpty(self):
          return True if len(self.queue) == 0 else False
    
      def size(self):
          return len(self.queue)
  
      # Object To String Override
      def __str__(self):
          return str(self.queue)
  ~~~

  - **연결리스트를 이용한 구현**

  > 큐의 경우 <u>동적배열을 지원하지 않는 언어</u> 의 경우 배열을 이용한 구현 시 `front` , `rear` 인덱스를 이용해 구현해야하기 때문에 **언어 의존적**
  >
  > 동적 배열을 지원하지 않는 언어의 경우 **크기 제한** 이 문제사항이 될 수 있으며 노드 개수만큼 메모리를 사용하기 때문에 **메모리 효율성** 에 유리함
  >
  > **구현**
  >
  > - `start` , `end` 두개의 노드 포인터를 이용해 큐의 시작과 끝을 체크
  > - `enQueue` 할 때 `end` 포인터에 새로운 노드를 연결
  > - `deQueue` 할 때 `start` 포인터의 노드를 반환 후 다음 노드를 새로운 `start` 로 교체
  > - `start` 포인터에 노드가 없다면 비어있음
  > - `start` 노드가 비어있을 때 `end` 포인터도 제거함으로서 **Double - Check** 소요를 줄임

  ~~~python
  class Node:
    def __init__(self, data):
      self.data = data
      self.next = None
  
  class NodeQueue:
      def __init__(self):
          self.start = None
          self.end = None
          self.error = "Error - Queue is Empty"
  
      def isEmpty(self):
          return True if self.start == None else False
  
      def enQueue(self, data):
          node = Node(data)
          if self.isEmpty():
              self.start = node
              self.end = node
          else:
              self.end.next = node
              self.end = self.end.next
  
      def deQueue(self):
          if self.isEmpty():
              return self.error
          data = self.start.data
          self.start = self.start.next
          # if after dequeue, queue is empty
          if self.start == None:
              self.end = None
          return data
      # fetch front data
      def front(self):
          if self.isEmpty():
              return self.error
          return self.start.data
  		# fetch rear data
      def rear(self):
          if self.isEmpty():
              return self.error
          return self.end.data
  
      def __str__(self):
          if self.isEmpty():
              return self.error
          # fetch all data in queue
          s = []
          node = self.start
          while node != None:
              s.append(node.data)
              node = node.next
          return ' > '.join(map(str, s))
  ~~~

  