### node.js ) Fetch vs Axios. Why i use axios ?

>✔ **Lang: Typescript**
>
>✔ **Platform: Node.js**



#### Fetch

- `fetch` - JS 의 서버와의 네트워크 통신을 지원해주는 **내장 Promise 베이스 비동기 HTTP client API**

  > **알아두기) ** node.js 는 기존 JS 에서 **Browser** 기능을 지원하지 않으므로 `fetch` 를 기본 API 로서 지원하지 않아 `node-fetch` 를 이용함
  >
  > `node-fetch` - [Github Link](https://github.com/node-fetch/node-fetch)

- React, RN 등에서는 내장 API 이므로 의존성이 낮아 **현재 프레임워크 버전에 대한 호환 업데이트를 신경쓰지 않아도 됨**

  > **알아두기) ** node.js 는 내장 API 가 아니므로 해당하지 않음

- `timeout` 미지원으로 네트워크 에러가 발생해 응답이 지연될 때에 대한 사용자의 커스텀 핸들링이 필요 *( 예시 코드 )*

~~~javascript
function fetchWithTimeout(targetURL, fetchOptions, timeout = 2000) {
    return Promise.race([
        fetch(targetURL, fetchOptions),
        new Promise((_, reject) =>
            setTimeout(() => reject(new Error('timeout occured !')), timeout)
        )
    ]);
}
~~~

- `Request Aborting` 지원 - **2020. 4. 5 패치로 모든 브라우저에 지원 ( IE 만 제외 ) ** *( 예시 코드 )*

  > **AbortController** 는 Scalable 하게 설계되어 있으므로 `timeout` 등에 대한 구현도 가능할 것 같음
  >
  > Document 를 통한 **AbortController** 의 이해 및 숙지가 필요함
  >
  > **업데이트 시점 ~ 2021. 4. 16 까지의 다양한 블로그 글 등에서 해당 부분의 미지원이 단점이라고 기술해놓았으나 이는 잘못된 정보임** 

~~~javascript
const controller = new AbortController()
const signal = controller.signal

function fetching() {
  var urlToFetch = "http://hubtwork.com/articles/3";
  fetch(urlToFetch, {
    method: 'get',
    // controller 의 abort() 함수 실행시 시그널이 토글되어 에러를 발생시키며 요청을 중단함
    signal: signal,
  })
    .then(function(response) {
    console.log('Fetch complete');
  }).catch(function(err) {
    console.error(` Err: ${err}`);
  });
}
// Aborting Signal 을 보낼 함수 구현
function abortFetching() {
  console.log('Now aborting');
  controller.abort()
}
~~~

- `Error Handling` 지원 *( 예시 코드 )*

~~~js
fetch(url)
  .then((response) => {
    if (response.ok) {
      return response.json()
    } else {
      throw new Error('network error')
    }
	})
  .then((data) => {
  	// 응답 처리
  	...
	})
  .catch((error) => {
  	// 에러 처리
  	console.log(error)
	})
~~~



#### Axios

- `axios` - node.js 의 대표적인 **Promise 베이스 비동기 HTTP client API 모듈**

  > `axios` - [Github Link](https://github.com/axios/axios)

- 외부 라이브러리이므로 의존성이 높아 **현재 프레임워크 버전에 대한 호환 업데이트가 이루어졌는지**가 중요함

- `timeout` 을 지원해 네트워크 에러가 발생해 응답이 지연될 때에 대한 핸들링 가능 *( 예시 코드 )*

~~~javascript
const instance = axios.create({
  baseURL: targetUrl,
  timeout: 2000
})
~~~

- `Request Aborting` 지원 - `cancelToken` 을 이용해 구현할 수 있음 *( 예시 코드 )*

  > **AbortController** 보다 직관적이고 사전 이해의 의존도가 크게 높지 않다고 생각됨

~~~javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('http://hubtwork.com/articles/3', {
	// cancelToken 의 cancel 이 호출되면 토큰이 발행되어 오류를 발생시키며 요청을 중단함
  cancelToken: source.token
}).catch(function (thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message)
  } else {
    // Error Handling
  }
})

source.cancel('Operation canceled by the user.')
~~~

- `Error Handling` 지원 *( 예시 코드 )*

~~~javascript
axios
  .get('http://hubtwork.com/articles/3')
  .catch(function (error) {
  	// 전송된 요청에 대해 2xx 이외의 상태코드에 의한 응답
  	if (error.response) { ... }
    // 전송된 요청에 대한 응답이 누락됨
    else if (error.request) { ... }
    // 처리할 수 없는 오류가 발생했을 경우
    else { ... }
    // axios 가 제공하는 에러 핸들링 객체
    console.log(error.config)
	}
~~~



### Conclusion

---

#### JS ( Pure JS, React, Vue , ... etc ) 에서의 관점

- React, Vue 등 일반적인 JS 프레임워크에 대해서는 이전에 **axios** 만 편의성 제공 및 지원했던 `Request Aborting` , `Error Handling` 등이 최근에는 **fetch** 에서도 모두 지원되어 서비스의 기존 레거시 코드, 개발자의 습관 및 기호에 따라 원하는 것을 채택해 사용해도 무방함
- 보다 개발 편의성을 추구한다면 **axios** , 외부 모듈에 대한 **`dependency`** 를 낮추고자 한다면 **fetch** 를 사용하는 것이 좋음

#### node.js 에서의 관점

- **node js** 에는 **Browser** 기능이 지원되지 않으므로 **fetch 및 AbortController 등** 을 지원하지 않아 이를 코드화시키고자 한다면 결국엔 외부 라이브러리를 이용해야함
- **node js** 의 입장에서 보았을 때 양 쪽 모두 동일하게 외부 라이브러리를 불러와 사용해야한다는 점에 있어서 편의성과 코드 완성도는 엄청난 차이가 있다고 생각하며 이는 **axios** 가 우세함
- 하지만 단순 편의성 뿐 아니라 외부 라이브러리 이용시 모듈의 크기 또한 중요하며 이는 **node-fetch** 가 **axios** 의 1/3 정도로 가벼움
- 보다 개발 편의성을 추구한다면 **axios** , 다른 js 프레임워크로 만든 서비스의 코드에서 **fetch** 를 사용한다면 코드 재사용성을 높일 수 있는 **node-fetch** 를 사용해 구현하는 것이 좋음



### REFERENCE

- https://github.com/node-fetch/node-fetch
- https://github.com/axios/axios