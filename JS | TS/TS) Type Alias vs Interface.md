### TypeScript ) Type Alias vs Interface

>✔ **Lang: Typescript**



#### Type Alias

- `type` - 커스텀 타입을 정의해서 사용할 수 있음

~~~typescript
type Student {
		sId: number,
		name: string
}
const s1: student = {
  	sId: 1,
  	name: 'Hubtwork'
}
const s2 = {} as Student
s2.sId = 2
s2.name = 'Network'
~~~

- `type` 은 정의된 이후 재정의될 수 없음 ( 정의의 변경이 불가능 )

~~~typescript
type Student {
		sId: number,
		name: string
}
type Student {
		sex: string
}	// error TS2300: Duplicate identifier 'Student'.
~~~

- `type` 은 원시값, 유니온, 튜플 등 다양한 타입 선언을 지원함

~~~typescript
// Literal Union
type FirstName = 'kim' | 'lee' | 'heo' | 'park' | null
type DIce = 1 | 2 | 3 | 4 | 5 | 6
// Object Literal Union
type Identity = { age: number } | { name: string }
// Function Union
type RestrictFunction = (() => string) | (() => number) | (() => boolean)
// Type , Interface Union
type School = HighSchool | MiddleSchool | LowSchool
// Tuple
type Dictionary = [string, any]
~~~

- `type` 은 에러 메시지에 이름이 표현되지 않음



#### Interface

- `interface` - JS 를 개선해 TS 로 전환하면서 인터페이스 개념이 추가됨
- 인터페이스와 관련된 다양한 기능이 제공되지만 **타입 정의**에도 사용할 수 있음

~~~typescript
interface Student {
  	sId: number,
		name: string
}
const s1: student = {
  	sId: 1,
  	name: 'Hubtwork'
}
const s2 = {} as Student
s2.sId = 2
s2.name = 'Network'
~~~

- `interface` 는 정의된 이후 재정의될 수 있음 ( 정의의 변경이 가능 )

~~~typescript
interface Student {
		sId: number,
		name: string
}
interface Student {
  	sex: string
}
const a: Student = {
  sId: 13,
  name: 'hi',
  sex: 'f'
}
~~~

- `interface` 는 상속 및 구현을 통한 내용의 확장이 가능함

~~~typescript
// extends
interface School { }
interface HighSchool extends School {}
interface LowSchool extends School {}
// implements
class HighSchoolBuilder implements HighSchool {}
~~~

- `interface` 는 에러메시지에 이름이 표현됨



#### Conclusion

- `type` 은 유니온, 튜플, 혹은 명시적인 파라미터 / 리턴 타입을 정의할 때 사용할 것

- `interface` 은 상속, 구현 등을 통해 확장 가능한 객체의 형태를 정의할 때 사용할 것

- **사용에 대한 개인적 견해**
  - `type` - 명시적으로 사용자 정의 타입 쌍을 만들거나 파라미터 / 리턴 타입에 대해 엄격한 룰을 적용하고 싶을 때
  - `interface` - 객체의 형태를 나타내는 추상체 또는 확장 가능한 타입을 정의하고 싶을 때



### REFERENCE

- https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces