### Kotlin ) Data Class

> ✔ **Lang : Kotlin**



#### Data Class

- `Data Class` - **Kotlin** 에서 데이터만을 보관하기 위해 지원되는 클래스

- **Kotlin** 의 고유 클래스 중 하나로 데이터를 다루기 위한 다양한 편의성을 제공함

- **`Data Class` 의 특징**

  - `equals()` / `hashCode()` / `toString()` 자동 생성

    > 위 자동 생성되는 함수들은 모두 주 생성자 외의 프로퍼티에는 적용되지 않음

  - **Destructuring Declarations** - 자체적 `componentN()` 함수 제공

  - `copy()` 함수 제공

  - `interface` 상속 가능

- **`Data Class` 의 제한사항**

  - 기본 생성자에 최소 하나 이상의 매개 변수가 있어야 함

  - 기본 생성자의 파라미터는 `val` , `var` 로 이루어져야 함

  - `abstract` , `open` , `sealed` , `inner` 속성 사용 불가

  - **`Data Class` 상속 불가능**

  - **`abstract class` , `interface` 상속 가능**

    > 상속할 시 파라미터를 **override** 함으로서 계층구조의 표현이 가능

  ~~~kotlin
  abstract class School(open val schoolNo: Int)
  data class ElementarySchool(
    override val schoolNo: Int,
    val schoolName: String
  ): School(schoolNo)
  data class HighSchool(
    override val schoolNo: Int,
    val schoolName: String,
    val schoolArea: String
  ): School(schoolNo)
  ~~~

- **`Data Class` 의 형태**

  ~~~kotlin
  data class Student(
    val studentId: Int = 0,
    val name: String = "",
    val major: String = ""
  )
  ~~~



#### copy() 함수 ?

- 기존 데이터에 대해서 특정 필드의 값만 변경해 사용하고 싶을 때 `copy()` 를 이용함

- **이용 )** 변경할 필드에 대해서만 파라미터를 이용해 재정의

  ~~~kotlin
  fun makeNewStudentWithSameMajor(student: Student, sId: Int, sName: String): Student {
    val newStudent: Student = student.copy(studentId = studentId, name = sName)
    return newStudent
  }
  ~~~



#### Destructuring Declarations

- `Data Class` 는 내부적으로 **componentN** 이 생성되어 각 프로퍼티에 자동으로 대입

  ~~~kotlin
  val student: Student = Student(1227, "hubtwork", "CS")
  val (studentId, name, major) = student
  ~~~

- 위 코드는 아래 코드와 같이 컴파일됨

  ~~~kotlin
  val studentId = student.component1()
  val name = student.component2()
  val major = student.component3()
  ~~~

  

### REFERENCE

- https://kotlinlang.org/docs/destructuring-declarations.html
- https://kotlinlang.org/docs/data-classes.html#copying

