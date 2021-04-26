### node.js ) Dotenv. Conceal your ApiKeys.

>✔ **Lang: Typescript**
>
>✔ **Platform: Node.js**



#### Background

- 서버 프레임워크인 **node.js** 를 이용한 개발을 진행하다보면 **DB 정보, port 정보, 외부 Api 키** 등 서비스 전체에서 필요하면서 숨겨야하는 전역 상수들이 존재함
- **github** 와 같은 외부 레포지토리를 이용해 형상관리를 진행하는 경우, 이는 외부에 절대 공개되어서는 안 되기 때문에 **`.gitignore`** 등을 통해 외부 레포지토리에는 공개하지 않으면서 전역 상수들을 일괄 관리할 수 있는 방법이 필요함



#### Dotenv ?

- **`Dotenv`** - `.env` 파일의 환경변수 key / value Pair 를 **`process.env`** 로 일괄 로드하는 패키지

- **Installation**

  > **npm install** 을 이용해 node.js 프로젝트에 설치

  ~~~bash
  npm install --save dotenv
  ~~~

- **Usage**

  >프로젝트의 루트 디렉토리에 `.env` 파일 생성 후 필요한 환경변수를 작성
  >
  >각 환경변수는 **Key=Value** 형태로 작성

  ~~~
  SERVER_PORT=3000
  
  DB_PORT=1004
  DB_HOST=localhost
  DB_USER=user
  DB_PASSWORD=pass
  
  A_API_KEY=apiKey
  A_API_SECRET=apiSecret
  ~~~

  > 아래와 같이 **dotenv** 를 import 해 환경변수들을 로드하고 **`process.env`** 의 멤버로서 Key 에 접근해 환경변수 값을 사용

  ~~~javascript
  require('dotenv').config()
  ...
  const apiClientKey = process.env.A_API_KEY
  const apiClientSecret = process.env.A_API_SECRET
  ~~~

