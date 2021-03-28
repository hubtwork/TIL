### Data Structure - Deque



#### Deque

- `Deque` - **양방향 입출력** 이 가능한 **Stack + Queue** 형태의 선형 자료구조
- **Deque** - `Stack` 과 `Queue` 의 결합
  - **양방향 입출력** 을 통해 데이터 삽입 / 삭제가 편하며 <u>가변적인 크기</u> 의 자료구조
  - **시퀀스 컨테이너** 의 형태이기 때문에 랜덤 접근이 가능함
  - **삽입, 삭제, 탐색** 모두 시간복잡도 `O(n)` 을 가짐
- **Deque Operation**
  - `addFront` - 데크의 처음에 데이터 삽입
  - `addRear` - 데크의 마지막에 데이터 삽입
  - `removeFront` - 데크의 첫 데이터 삭제
  - `removeRear` - 데크의 마지막 데이터 삭제
  - `size` - 현재 데크 내의 데이터 개수를 반환
  - `isEmpty` - 데크가 비었는지 여부를 반환 ( `size` == 0  *or* `front` == `rear` )

<p align="center"><img src="../../assets/img/deque.png" alt="Imgur" width="400" /></p> 

- **Deque Implementation**

  - **배열을 이용한 구현**

  > 구현이 간편하며 **python** 과 같이 동적 배열을 지원하는 언어의 경우 크기에 제한을 받지 않아 구현에 제약사항이 없음
  >
  > **python** 의 경우 `addFront()` 와 `removeFront()` 를 List 의 `insert(0, T)` 와  `pop(0)` 을 이용해 구현할 수 있지만 그 뒤 인덱스들의 모든 데이터를 한칸씩 밀거나 당겨오는 연산을 수반하기 때문에 **시간복잡도 O(N)** 이 추가됨

  ~~~python
  class Deque:
      def __init__(self):
          self.deque = []
          self.error = "Error - Queue is Empty"
      
      def addFront(self, data):
          self.deque.insert(0, data)
          
      def addRear(self, data):
          self.deque.append(data)
      
      def removeFront(self):
          if self.isEmpty():
              return self.error
          return self.deque.pop(0)
    
      def removeRear(self):
          if self.isEmpty():
              return self.error
          return self.deque.pop()
  
      def front(self):
          if self.isEmpty():
              return self.error
          return self.deque[0]
    
      def rear(self):
          if self.isEmpty():
              return self.error
          return self.deque[1]
    
      def isEmpty(self):
          return True if len(self.deque) == 0 else False
    
      def size(self):
          return len(self.queue)
  
      # Object To String Override
      def __str__(self):
          return str(self.deque)
  ~~~

  - **Python Deque**

  > 위에서 말한 **배열** 을 이용한 구현에서 연산 후 데이터 재배열에 의한 시간 지연 문제를 내장 모듈 **Deque** 를 이용해 해결할 수 있음
  >
  > **Deque** 의 기능뿐 아니라 리스트의 중간 데이터 삽입 / 삭제 드을 지원함
  >
  > **Module** - `collections.deque`

  ~~~python
  from collections import deque
  # init deque with list
  q = deque([])
  
  # add Operation
  q.appendleft(0) # addFront
  q.append(1) # addRear
  
  # remove Operation
  q.popleft() # removeFront
  q.pop() # removeRear
  
  # extend Operation
  q.extendleft([-3, -2, -1]) # extendFront
  q.extend([2, 3, 4]) # extendRear
  
  # insertion / removal in list
  q.inert(4, 1) # Insert(idx, T)
  q.remove(1) # remove(T) :: delete T which first scanned from inorder index
  
  # Reverse Rearrangement
  q.reverse()
  ~~~

