---
title: 726
typora-copy-images-to: dynamoose
tags:
  - dynamoose
  - dynamodb
  - indexes are not synchronized and update flag is set to false
---

# Trouble shooting

### dynamoose

#### indexes are not synchronized and update flag is set to false

서버와 로컬 스키마가 다를 때 `update` 가 `false` 로 설정되어 스키마를 업데이트 할 수 없다는 뜻이다. `false` 가 의도된 것이라면 아래와 같은 이슈 일 수 있다.

```javascript
dynamoose.setDefaults({
  create: false,
  update: false,
})
```

이와 같이 기본 설정이 들어간 경우라고 하면 이 이전에 `dynamoose.model` 등을 호출한 경우 문제가 된다. 순서에 종속성이 강하므로 이를 먼저 확인한다.



### Next.js

#### Universal config

유니버살 컨피그라는 개념이 있는데 서버에서와 동일하게 클라이언트에서도 환경 변수 등에 접근할 수 있도록 하는 것이다.

이를 가능하게 하는 유사한 친구로는 *webpack* 의 `DefinePlugin`, `EnvironmentPlugin` 이 존재한다. 이는 **컴파일** 타임에 환경변수 주입이 가능하다.

*Next.js* 같은 경우는 SSR를 지원하는데 서버가 실행되는 런타임 

next/config`  



#### AWS_IAM

> Failed to load https://api.dev.readi.sh/user/ap-northeast-2:9e60248f-6c6c-4d24-afc1-353d2df3b77c: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access.

*API Gateway* 통과 못한 상태. `role`, `role policy` 지정 필요.

> index.js:2178 Error: Network Error     at createError (createError.js:16)     at XMLHttpRequest.handleError (xhr.js:87)

에러시 `@aws-amplify/api` 에서 뱉는 에러

*API Gateway* 인증을 `AWS_IAM` 으로 할 때 반드시 필요

##### ~~role~~

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "apigateway.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

필요 없는 것으로 파악됨

##### role policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "execute-api:Invoke", // 확인됨
        "lambda:InvokeFunction", // 필요 없는 것으로 파악 됨
        "lambda:InvokeAsync"  // 필요 없는 것으로 파악 됨
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

### Github

Github App과 OAuth는 꽤나 다른면이 있어서 섯불리 건들면 한달 날아간다. 권한에 대한 차이가 있으며 Github App 토큰과 Github App OAuth 토큰은 권한이 다르며 Github App OAuth 토큰과 OAuth 토큰도 가질 수 있는 권한의 차이가 있다.

- 예를 들어 Github App 토큰으로는 유저의 개인정보에 접근할 수 없다.
- Github App OAuth 토큰은 개인 정보를 가질 수 있으나 Webhook 등을 생성할 수 없다.
- 가장 오래된 OAuth 는 제약이 거의 없는 것으로 보인다.

Github App 은 싱글 엔트리포인트로 모든 이벤트를 받아서 처리할 수 있는 App 개념이다. 개념 자체는 좋다. 근데 이를 가지고 실험적인 모험을 하기에는 퍼미션 늪의 위험성이 존재한다.

또한 Github App 인스톨레이션을 통해 아이덴티디가 확보된다는 점이 특징이고 OAuth는 유저의 권한을 행사한다.

이제 보니 그런거 같다… 그런거같다. :sob:

### TypeScreipt

#### definition을 중복해서 정의할 경우 어시스트에서는 첫번째 매칭부터 처리해서 보여준다.

잘못된 예

```typescript
interface Receiver extends EventEmitter {
  on(event: ReceiverEvents, handler: (body: any) => void): this
  on(event: ReceiverEvents.Ping, handler: (body: GithubEvent.PingEvent) => void): this
  on(event: ReceiverEvents.Push, handler: (body: GithubEvent.PushEvent) => void): this
}
```

잘된 예

```typescript
interface Receiver extends EventEmitter {
  on(event: ReceiverEvents.Ping, handler: (body: GithubEvent.PingEvent) => void): this
  on(event: ReceiverEvents.Push, handler: (body: GithubEvent.PushEvent) => void): this
  on(event: ReceiverEvents, handler: (body: any) => void): this
}
```

#### noImplicitAny

이게 `false` 설정되지 않으면 타입 데피니션이 없는 모듈을 에러를 내게된다.

### NPM, Yarn

#### Link

`npm link` 는 커맨드 라인 유틸인 경우 `node_modules/.bin/` 에 실행 링크까지 생성해주지면 `yarn link` 는 그저 `node_modules` 에 링크를 띄워주는 것으로 보인다

### Babel

>Requires Babel "^7.0.0-0", but was loaded with "6.26.3". If you are sure you have a compatible version of @babel/core, it is likely that something in your build process is loading the wrong version. Inspect the stack trace of this error to look for the first entry that doesn't mention "@babel/core" or "babel-core" to see what is calling Babel. (While processing preset: "/Users/bglee/workspace/src/github.com/deptno/api-fakeway/node_modules/@babel/preset-env/lib/index.js")

```sh
 yarn add -D 'babel-core@^7.0.0-0'
```

```javascript
const hello = 'world'
const second = 23
```
