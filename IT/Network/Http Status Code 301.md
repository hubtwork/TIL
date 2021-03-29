### Http Status Code 301 Handling



#### Background

- `모바일 애플리케이션` 에서 API 서버에 **Api Request** 를 요청
- `status code 301` 반환과 함께 서버상에서 **Redirection** 이 되어야 하나 지속적으로 잘못된 `url` 이 반환됨
- 계속해서 잘못된 `url` 에 대해 **Api Request** 를 요청하여 실패하지만 구현상의 문제는 없음



#### Http Status Code 301

- `status code 301` - **Moved Permanently** 를 의미하는 상태코드. `Redirection` 할 URL 을 반환.

- **Action** - `애플리케이션` 은 **301** 에 의해 반환된 `Redirection URL` 으로 재요청

- **Problem** 

  - `서버` 가 **301** 에 의해 `Redirection URL` 을 반환하면 그 생존주기는 `서버` 의 생존주기와 같음

  - `애플리케이션` 코드에서 구현된 `Target URL` 이 아닌 `Redirection URL` 로 요청하고 이는 실패할 수 있음

  - 잘못된 요청과 응답 실패로 인해 `애플리케이션` 에서 오류가 발생하나 프록시를 통한 확인 외엔 `Redirection URL` 및 요청을 확인할 수 없음

- **Solution**

  - `status code 302` - **Found** 상태코드를 이용해 일시적인 `Redirection URL` 반환 및 `GET` 메소드로 재요청
  - `status code 307` - **Redirect Temporarily** 상태코드를 이용해 `Redirection URL` 반환 및 동일한 메소드로 재요청

- **Summary**
  - 서버애플리케이션에서 `Redirection` 이 필요할 때
    - **GET** 요청이라면 `status code 302` 를 사용해도 됨
    - **POST** 요청과 같이 **GET** 이 아닌 요청일 경우 `status code 307` 을 이용해 구현할 것



### REFERENCE

- https://perfectacle.github.io/2017/10/16/http-status-code-307-vs-308/

