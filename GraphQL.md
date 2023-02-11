> GraphQL은 Facebook에서 만든 API용 쿼리 언어이다.
> 클라이언트는 API 서버에 GraphQL 쿼리를 전송해 내가 필요한 정보를 선택해 가져올 수 있다.

## GraphQL과 REST API

웹 개발을 처음 시작하면 REST API를 먼저 접하게 된다. REST API에서 클라이언트는 API의 엔드포인트를 이용해 데이터를 주고받는다.
GraphQL의 경우에는 엔드포인트가 한개만 존재하며, 쿼리를 통해 데이터를 주고받는다. 
쿼리(query)의 사전적 의미는 "질문"이다. 클라이언트가 API에 특정 값을 줄 수 있냐고 질문을 하면 API는 해당 값에 대한 값을 반환한다.

### GraphQL 쿼리문과 응답 값 예시

![](https://velog.velcdn.com/images/kisuk623/post/1132bcac-68b4-4fa7-a34c-ad3f01f16477/image.png)


### GraphQL의 장점

#### UnderFetching, OverFetching이 일어나지 않는다

Under / OverFetching은 필요한 데이터를 한 번에 받지 못하거나, 필요한 데이터보다 많은 데이터를 받게 된다는 것이다.

velog에서 포스트를 다 읽고 나면 하단에 관심 있을 만한 포스트의 리스트가 나오는데, 이는 포스트의 내용과는 별개의 정보가 필요하다. 한 번의 요청에 필요한 데이터를 모두 받지 못해 UnderFetching이 일어난다.

다른 기능의 페이지라도 같은 엔드포인트를 이용해 데이터를 가져올 경하지 않는 경우가 있다. 첫 번째 페이지에서는 유저의 이름이 화면에 보여야 하지만, 두 번째 페이지에서는 이름이 필요 없는 경우, 두 번째 페이지에서는 OverFetching이 일어난다.

GraphQL을 사용하면 한 번의 요청에 두 가지 이상의 데이터를 가져올 수 있고, 내가 원하는 값만 가져올 수 있어 데이터의 사이즈가 감소해 네트워크의 낭비를 줄일 수 있다.

### GraphQL의 단점

#### 파일 업로드의 불편함

REST API에서 클라이언트에서 `form-data`형식을 이용해 파일을 업로드 한다. 하지만 GraphQL은 `form-data`를 지원하지 않아 기타 패키지를 이용해 업로드를 해야 한다.

#### 기본으로 지원하는 데이터 타입이 적다

GraphQL의 필드값들은 타입을 가지고 있어야 하는데, 최하단의 값들은 기본적으로 제공되는 Scalar타입을 이용해 타입을 정의한다. Scalar 타입에는 `Int`, `Float`, `String`, `boolean`, `ID`가 있는데 기본적으로 제공하지 않는 날짜 같은 경우에는 직접 Scalar 타입을 생성해 타입을 지정해 줘야 한다.


## Nest.js에서 GraphQL

Nest.js에서 GraphQL을 사용하려면 GraphQL모듈과 apolloServer모듈을 설치해야한다.

```bash
$ npm i @nestjs/graphql @nestjs/apollo graphql apollo-server-express
```

Nest.js에서 Express가 아닌 fastify를 사용할 경우 `apollo-server-express`대신 `apollo-server-fastify`를 설치하면 된다.

설치가 끝나면 app.module.ts에 GraphQL모듈을 import해준다.
```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      playground: false,
    }),
  ],
})
export class AppModule {}
```

playground는 GraphQL의 쿼리를 테스트, 타입 값 등을 확인할 수 있는 테스팅 예시 공간이다.
개발 단계에서 유용하게 사용되며, 배포단계에선 해당 옵션을 필수로 `false` 로 바꿔줘야 한다.
배포단계에서 playground가 `true`일 경우 의도치 않은 API에 대한 공격이 들어올 수가 있다.

### Playground 테스팅 예시

![](https://velog.velcdn.com/images/kisuk623/post/13352842-e101-4792-9eee-643862f8c88f/image.png)


### Schema

GraphQL의 쿼리는 Schema가 있어야 작동할 수 있다. Schema란 자료의 구조, 자료의 표현방법, 자료간의 관계를 정의한 구조로써 주로 데이터베이스에서 사용하며, GraphQL에서 Schema를 생성하는 방법은 Schema first, Code first 두 가지가 있다.

Schema first는 schema를 먼저 정의하고 그 뒤에 코드를 작성한다. 해당 schema에는 타입이 정의되어 있기 때문에 중복된 타입 지정을 작성하지 않아도 된다. 

Code first는 클래스를 선언한 뒤 타입을 지정해 주면 schema를 자동으로 생성시켜 준다.

#### schema 예시

![](https://velog.velcdn.com/images/kisuk623/post/e15ffb75-0661-4e35-a214-6b6c7e8f2fd1/image.png)

