### JS ) Object 객체 정보 Logging 시 주의할 점



#### Background

- 배포된 앱의 모니터링, 개발 중의 디버깅 혹은 테스트 등을 진행하면서 객체가 의도한 대로 핸들링되었는지, 요청 / 응답 되었는지 확인하기 위해 **로깅** 은 매우 중요함
- **JS** 의 경우 **`console.log()`** 를 이용해 콘솔 로깅하거나 **node.js** 의 경우 **`winston, morgan`** 과 같은 로깅 모듈을 이용해 로깅을 진행함



#### Problem

- **node.JS** 로 서버 개발을 진행하면서 **`callback, async/await`** 등을 이용한 비동기 함수를 작성하다보니 개발, 테스트 중에 각 시점(함수단위) 에서의 객체 추적을 통해 구현의 작동을 확인해야할 필요가 있음

- 필자는 주로 *typeof* , *instanceof* , *Class Instance* , *Json Object* 등을 **winston.logger** 를 이용해 출력하고 확인하는데, 이 때마다 `[object Object]` 혹은 `object` 로 출력됨

  > **주의 )** `console.log` 의 경우 객체에 대한 **참조** 를 로깅하기 때문에 **JS** 를 이용해 개발할 때 **document** 를 이용해 확인할 경우, 로그의 객체정보가 찍힌 로그와 다른 경우가 발생할 수 있다. 이에 **lodash** 의 `deepcopy` 를 이용해 안전하게 로깅하는 방안 등이 필요하다.



#### Analyze

- **JS** 는 자바나 다른 언어와 달리 객체에 대해 **Object.prototype.toString()** 을 재정의해주지 않기 때문에 사용자 지정 객체의 경우 해당 메소드의 재정의를 명시적으로 선언해주지 않으면 디폴트값인 **`[object Object]`** 를 반환

  > 출처 : [Javascript Object.prototype.toString 도큐멘트](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)



#### Solution

- **임시 방편 )** `JSON.stringify()` 를 이용해 객체를 **JSON string** 으로 변환해서 출력

- **정공법 )** `Object.prototype.toString()` 재정의 ( 모든 객체에 대해 작성하기 힘들어 제한적 )

  ~~~typescript
  class Student {
    name: string
    studentId: number
    constructor(name: string, studentId: number) {
      this.name = name
      this.studentId = studentId
    }
  }
  Student.prototype.toString = function studentToString(): string {
    return `[ studentId : ${this.studentId} ] name : ${this.name}`
  }
  ~~~

  

