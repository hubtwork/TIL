### Jest 를 이용한 테스트 중 MockClear 이용시 문제점



#### Problem

- **node.js** 패키지 개발 중 `axios` 기반의 Http Request, 서비스 에러 등을 핸들링할 수 있는 **ApiClient** 클래스를 설계 및 구현

-  **ApiClient** 기반의 **다양한 외부 API 대응 서비스** 들의 기초 구현 및 테스팅 후 세부 구현 방향으로 진행

- 각 서비스들의 **Mock class** 를 만들고 **jest** 모듈을 이용해 `axios` 를 **mocking** 해서 테스트 진행

- `axios` 의 응답 및 에러를 **Promise** 의 `resolve , reject` 를 이용해 각 테스트케이스 구현

- 하지만 **beforeEach()** 에서 **mockFn.mockClear** 를 실행하도록 구현했으나 이전의 테스트에서 **mocking** 한 Response 가 수행되는 문제 발견

  ~~~javascript
  const axios = require('axios')
  jest.mock('axios')
  describe('testsuite', () => {
    ...
    beforeEach(() => axios.mockClear())
    ...
    // test 1 - test service error
    test('test1', () => {
      axios.mockImplementationOnce(() =>
        Promise.reject()
      )
    ...
    })
    
    // test 2 - test error response
    test('test2', () => {
      axios.mockImplementationOnce(() =>
        Promise.reject({
        	response: {
            status: 400
          }
      	})
      )
      // Test Error. ( expected test2's reject, but test1's reject come in. )
    ...
  ~~~

  

#### Tracking

- **외부 API 대응 서비스들** 의 경우, 서비스 단에서 직접 요청을 보내기 전 파라미터 검증, API 지원 여부 검증 등 API 서비스 별로 이용에 문제가 없는지 확인함
- 만약 문제가 있을 시, **Service Error** 를 발생시켜 **ApiClient** 를 호출하지 않고 사용자에게 **Error Response** 를 전달
- 즉, 매 테스트 이전 **mocking** 된 `axios` 를 **`mockFn.mockClear()`** 을 이용해 초기화하였으나 **Service Error** 를 테스트한 이전 테스트에서 **mocking** 한 `axios` 의 **mock response ( Promise.reject )** 가 호출되지 않음
- 이에 실제 서버의 **Error Response** 를 테스트하는 다음 테스트에서 기존의 실행되지 않았던 `axios` 의 **mock response ( Promise.reject )** 가 응답으로서 처리됨을 확인



#### Analyzing

- **jest** 의 `mockFn.mockClear()` 을 확인한 결과 **mocking 된 함수** 를 정리하는 방법이 처리 수준별로 다음 3가지가 존재함

- **`mockFn.mockClear()`** - `mockFn.mock.calls` 와 `mockFn.mock.instances` 를 초기화함

  > `mockFn.mock.calls` - **mockFn** 이 호출됫을 때의 매개변수 목록
  >
  > `mockFn.mock.instances` - **mockFn** 이 생성자로 호출됬을 때 생성한 인스턴스 목록

- **`mockFn.mockReset()`** - `mockFn.mockClear` 의 기능과 더불어 **mocking** 되는 시점으로 되돌림 ( `mockFn` 은 `undefined` 로 초기화됨 )

- **`mockFn.mockRestore()`** - `mockFn.mockReset` 의 기능과 더불어 **`jest.spyOn()`** 등을 통해 일부 메서드만 **mocking** 해 다른 함수로 치환한 경우, **mockFn** 을 undefined 로 초기화

  > `jest.fn` 을 이용해 **mocking** 한 경우, **mockRestore()** 로도 초기화 할 수 없음.
  >
  > 따라서 객체의 특정 메서드만을 **mocking** 하고 싶은 경우, **jest.spyOn()** 을 이용하고 매 테스트가 끝날 시 **mockRestore()** 로 초기화하는 것이 보다 테스트의 무결성을 보장할 수 있음.
  >
  > **주의 )** **`mockFn.mockRestore()`** 는 `jest.spyOn()` 에 의해 생성된 **mock** 에 대해서만 작동함



#### Solution

- 즉, 매 테스트 이전 **mockClear()** 를 호출 시 아래와 같은 문제가 발생함
- 서비스로직에 의해 **mocking** 한 `axios` 를 호출하기 이전 시점의 응답에 대해 처리하고 테스트가 종료되었고 실제 **mocking** 한 `axios` 를 호출한 시점에선 이전의 반환되지 않은 **mockFn** 이 반환됨

- 즉, 이전 테스트에서 **mock Response** 가 명확히 **mockClear()** 가 아닌 **mockReset()** 수준의 **free Mock** 전략이 필요함

  > `jest.spyOn()` 을 이용한 **mocking** 이 아닐 때는 **mockRestore()** 수준의 전략은 불필요함 ( 이를 구분해 두는 것이 코드의 명확성을 높이는 것에 더 좋다고 생각함 )



### REFERENCE

- https://jestjs.io/docs/mock-function-api#mockfnmockclear
- https://github.com/facebook/jest/blob/bc50e7f360ab1845abbaa0b3ad788caead0d3174/packages/jest-mock/src/index.ts

- https://github.com/facebook/jest/issues/5143