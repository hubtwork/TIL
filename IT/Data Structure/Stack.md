### Data Structure - Stack



#### Stack

- `Stack` - **LIFO ( Last In First Out )** 형태의 선형 ( Linear ) 자료구조

- **LIFO** - 마지막에 들어온 데이터가 가장 먼저 나가는 형태로서 **후입선출** 이라고도 함

  - 예시로 **텍스트 에디터** 의 `history management` 를 들 수 있다
  - 작성한 내용의 변화 등은 **LIFO** 형태로 저장되어 사용자가 `undo` / `redo` 의 기능을 수행할 수 있도록 한다

- **Stack Operation**

  - `push` - 스택에 데이터 삽입
  - `pop` - 스택의 마지막 데이터 삭제
  - `peek` - 스택의 마지막 데이터 반환
  - `isEmpty` - 스택이 비었는지 여부를 반환
  - `size` - 스택의 크기 반환

- **Stack Implementation**

  - **배열을 이용한 구현**

  > 구현이 간편하며 **python** 과 같이 동적 배열을 지원하는 언어의 경우 크기에 제한을 받지 않아 제약사항이 없음

  ~~~python
  class Stack:
    def __init__(self):
      self.stack = []
      
    def push(self, data):
      self.stack.append(data)
      
    def pop(self):
      return self.stack.pop()
    
   	def peek(self):
      return self.stack[-1]
    
    def isEmpty(self):
      return ( len(self.stack) == 0 )
    
    def size(self):
      return len(self.stack)
  ~~~

  - **연결리스트를 이용한 구현**

  > 동적 배열을 지원하지 않는 언어의 경우 **크기 제한** 이 문제사항이 될 수 있으며 노드 개수만큼 메모리를 사용하기 때문에 **메모리 효율성** 에 유리함

  ~~~python
  class Node:
    def __init__(self, data):
      self.data = data
      self.next = None
  
  class Stack:
    def __init__(self):
      self.stack = None
      
   	def push(self, node):
      node.next = self.stack
      self.stack = node
        
   	def pop(self):
      if self.stack == None:
        return "No Data In Stack"
      else:
        data = self.stack.data
        self.stack = self.stack.next
        return data
      
   	def peek(self):
      if self.stack == None:
        return "No Data In Stack"
      else:
        return self.stack.data
    
    def isEmpty(self):
      return True if self.stack == None else False
  ~~~

<p align="center"><img src="../../assets/img/stack.png" alt="Imgur" width="400" /></p> 

