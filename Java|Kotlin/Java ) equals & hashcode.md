### JAVA ) equals() & hashcode()



#### Background

- **자바** 를 사용해봤다면 `equals()` 와 `hashcode()`  두 함수를 수도 없이 오버라이딩해봤을 것이고 이를 놓쳐서 고생해본 경험이 있을 것이다
- 최근에는 **코틀린** 을 사용하면서 모델을 정의할 때 **data class** 를 자주 이용하게 되면서 그 중요성을 잊고 있었다
- 이를 정리하고 추후에 **왜 코틀린을 사용하면서 까먹게 되었는지** 를 비교 정리할 것이다



### Object Class

- `Object` - **자바** 의 최상위 클래스. 모든 클래스는 기본적으로 `Object` 클래스를 상속함

- 즉, `Object` 클래스는 모든 클래스가 공통적으로 가지고 있어야 할 기능들을 제공하는 클래스이다
- **`Object` 의 필수 Methods**
  - `equals()` - 객체가 동일한지 판단 **( 객체끼리 같을 수 있음 )**
  - `hashcode()` - 객체의 메모리주소의 해시값을 리턴 **( 객체마다 다름 )**
  - `toString()` - 객체를 문자열로 변환했을 때의 결과를 정의



#### equals()

- `equals()` - 인자로 넘어온 두 객체의 **내용** 이 같은지 확인함

- **`Object` 클래스의 `equals()` 정의를 살펴보자**

  - 두 객체의 주소값을 비교한 값을 리턴함

  ~~~java
  public boolean equals(Object obj) {
  		return (this == obj)
  } 
  ~~~

- 이를 변경하지 않고 사용하면 단순히 `==` 로만 비교하고 있기 때문에 <u>인스턴스 내의 값이 같더라도</u> 두 객체는 다른 것으로 간주된다

  - `==` 는 두 인스턴스가 **주소값이 같은지를** 판단한다
  - **주의 )** `==` 을 이용해 객체를 비교할 때는 되도록 `Primitive Data Type` 들에 대해서만 사용하자
  - `Primitive Data Type` - **자바** 가 기본적으로 제공하는 데이터 타입 ( `byte` , `int` , `double` 등 )

- **`equals()` Overriding**

  - 두 객체의 **주소값에 의한 동일성** 이 아닌 **논리적 동일성** 을 비교하기 위해 우리는 `equals()` 를 재정의할 수 있다
  - 아래 클래스의 경우 학생을 나타내는데 논리적으로 봤을 때 **학번이 같을 경우** 같은 학생을 가리킨다
  - 이를 위해 `equals()` 를 클래스에 맞춰 재정의해보자

  ~~~java
  class Student {
  		int studentNumber;
    	String name;
    	ArrayList<Course> registeredCourses;
    	...
    	Student(int studentNumber, String name) {
        	this.studentNumber = studentNumber;
          this.name = name;
        	this.registeredCourses = new ArrayList();
        	...
      }
    	...
   		@Override
    	public boolean equals(Object o) {
        	if (o == null) {
            	return false;
          }
        	if (o == this) {
            	return true;
          }
        	// 만약 대상이 Student 의 인스턴스가 아니면 False
        	if (!(o instanceof Student)) {
            	return false;
          }
        	// studentNumber 값이 같고, name 의 내용이 같으면 같은 객체로 판단
        	Student so = (Student)o
        	return so.studentNumber == studentNumber;
      }
  }
  ~~~

  - **논리적으로 보았을 때 같은 학생인 경우** 를 비교할 수 있도록 구현할 수 있다



#### hashCode()

- `hashCode()` - **Runtime** 중에 해당 인스턴스의 유일한 `int` 값을 반환하는 함수 ( **Object** 에서는 heap 에 저장된 메모리주소를 반환 )

- **`Object` 클래스의 `hashCode()` 정의를 살펴보자**

  - **JNI** 를 이용해 구현한 native 메소드로 객체의 해시코드를 반환함

  > **hashCode 규칙** ( Java 13 doc 주석 참조 )
  >
  > 1. 변경되지 않은 객체의 `hashCode` 호출 결과는 항상 같은 `integer` 이어야 한다
  > 2. `equals` 에 의해 같은 객체로 간주된다면 `hashCode` 호출 결과 또한 같은 값이어야 한다
  > 3. `equals` 에 의해 다른 객체로 간주되더라도 `hashCode` 가 다르지 않을 수 있다
  >    - 서로 다른 객체가 다른 `hashCode` 를 가질 경우 **해시테이블의 성능이 향상** 될 수 있다

  - 대놓고 `equals` 와 `hashCode` 는 **한 세트** 라고 설명하고 있다

  ~~~java
  public native int hashCode();
  ~~~

- **즉, `equals` 가 재정의되었다면 이에 맞춰 `hashCode` 또한 재정의하는 것이 좋다**

  - `equals` 를 재정의할 때 활용한 동일한 속성들을 이용해 `hashCode` 를 재정의하자
  - **< 이유 >** 
  - `equals` 재정의에 의해 논리적으로 같은 객체로 판단하고 `hashCode` 는 다른 객체가 있다고 가정하자
  - 두 객체는 같은 것으로 봐야함에도 불구하고 **HashSet, HashMap, HashTable** 등의 자료구조와 함께 사용할 때는 서로 다른 해시값을 반환하기 때문에 다른 위치에 저장된다
  - 즉, `equals` 에 의해 같다면 같은 `hashCode` 를 갖게 해 **동일한 Key** 로 간주될 수 있도록 하자
  - **주의 ) ** `hashCode` 를 재정의할 때 최대한 비교를 위한 핵심필드를 모두 포함해 *속도보단 성능* 을 챙기도록 하자

- **`hashCode()` Overriding**

  ~~~java
  @Override
  public int hashCode() {
    	int hashCode = 0;
    	// 일반적으로 해시코드 구현에 소수 31을 이용하지만, 나는 Lombok 라이브러리에서 해시코드를 위해 사용하는 59 를 사용한다
    	hashCode = 59 * hashCode + studentNumber;
    	return hashCode;
  }
  ~~~

  - **논리적으로 보았을 때 같은 학생인 경우** , 즉 학번이 같은 인스턴스의 경우 동일한 해시코드를 반환한다



#### SUMMARY

- `equals` 가 재정의되었다면 그에 맞춰 `hashCode` 도 재정의하자
- `equals` 와 `hashCode` 를 재정의할 때 같은 속성을 활용하자
- `equals` 의 결과는 속성이 변경되기 전에는 동일함을 보장할 수 있도록 하자



### REFERENCE

- https://docs.oracle.com/javase/10/docs/api/java/lang/Object.html