>API에서  유저를 판별하기 위해선 인증 절차를 거쳐 해당 유저를 검증하고 인가 작업을 통해 사용자에게 작업에 대한 접근 권한을 부여한다. 인증/인가엔 대표적으로 두 가지 방법이 있는데, 세션 기반 인증과 토큰 기반 인증이다. 토큰 기반 인증은 JWT토큰을 발급해 해당 토큰으로 유저에게 인증/인가를 해주는 것이고, 세션 기반 인증은 서버에서 사용자의 정보를 관리해 인증/인가를 해준다.
>오늘은 세션 기반 인증과 세션 인증에 주로 같이 사용되는 Redis를 알아보도록 하자.

## 인증/인가

웹 API 통신에 있어서 인증과 인가는 매우 중요한 역할을 한다. 클라이언트가 ID/PW입력을 통해 사용자의 정보 인증하면 서버는 해당 사용자를 검증하고, 세션(쿠키) 혹은 토큰을 발급해 준다.
인증을 받은 클라이언트는 추후 발생하는 API 호출에 대해 발급받은 쿠키 혹은 토큰을 같이 첨부해 전송하면 서버는 해당 정보를 받아 사용자의 권한을 부여해 API에 대한 값을 반환해 준다.

왜 번거롭게 쿠키를 받은 뒤 요청할 때마다 쿠키를 같이 전송해 줘야 할까? 
HTTP는 Stateless 특성이 있다. Stateless란 클라이언트가 이전에 전송한 상태를 보존하지 못한다는 것이다. 이전 상태를 보존하지 못해 클라이언트는 인증 여부를 알 수 없고, 서버 또한 보안을 위해 무차별적인 요청을 받아줄 수 없다.
이전 상태를 알지 못하더라도, 인증받은 세션과 쿠키를 통해 해당 사용자의 유효성을 판별할 수 있게 되고, 해당 정보를 통해 서버는 사용자에게 적절한 응답을 보내줄 수 있다.

### 토큰 인증

토큰 인증은 사용자가 로그인하면 서버는 해당 사용자의 인증정보를 담은 토큰을 발급해 클라이언트에게 반환한다.
클라이언트는 해당 토큰을 소유하고, 요청을 진행할 때마다 해당 토큰을 헤더에 첨부해 보내면 서버는 해당 토큰의 유효성을 검증하고 토큰에 기록된 사용자 정보를 통해 사용자를 판별한다.

### 세션 인증

세션 인증은 유저가 로그인을 하면 서버는 인증 정보를 서버에 저장하고, 클라이언트에 해당 사용자의 고유한 세션ID를 부여한다. 클라이언트는 세션ID를 쿠키로 이용해 브라우저에 저장하고, 이후 요청할 때마다 쿠키를 헤더에 같이 첨부해 전송하면 서버는 해당 쿠키를 받아 저장된 세션 ID의 유효성을 검증하고 서버에 기록된 유저 정보를 불러와 유저를 판별한다.


### 토큰 인증과 세션 인증의 차이

토큰 인증과 세션 인증의 가장 큰 차이점은 인증정보와 유저 정보의 저장하는 공간이 다르다는 점이다. 
토큰 인증방식은 서버에서 발급한 토큰을 클라이언트가 받아 브라우저에 저장한다. 토큰에는 인증정보와 서버에서 추가한 유저정보가 포함되어 있으며 인가 작업 시 서버는 해당 토큰의 유효성만 검증한다.

세션 인증방식은 서버에서 인증정보와 유저정보를 모두 가지고 있고 세션 ID를 발급해 유저 정보를 구별한다. 클라이언트에는 발급한 세션 ID를 쿠키로 브라우저에 저장한다. 클라이언트가 가지고 있는 쿠키에는 아무런 정보가 없고, 인가 작업 시 서버는 해당 쿠키를 받아 서버 내에 저장된 세션 ID와 비교해 유효성을 검증하고 유저정보를 획득한다.

이처럼 토큰 인증 방식과 세션 인증 방식의 차이점은 인증정보의 위치(클라이언트, 서버)에 따라 나뉜다.
토큰 인증 방식의 경우 서버에는 토큰 정보를 저장하지 않고 해당 토큰이 유효한 토큰인지 검사한다. 이 말은 혹여나 토큰이 탈취당했을 때 서버는 해당 토큰의 기능을 정지시키지 못한다는 것이다. 탈취당한 토큰은 유효기간이 끝날때 까지 서버에 정상적인 접근 권한을 가지게 된다.
세션인증 방식은 서버에서 인증정보를 가지고 있다. 클라이언트가 가지고 있는 건 서버의 세션 ID만 가지고 있고 유저 정보가 서버에 저장되어 있어 보안 측면에서 유리하며, 세션 ID가 탈취당했더라도 서버는 해당 세션ID의 인증정보를 삭제하면 더 이상 해당 토큰으로 인가 작업을 받지 못하게 된다.

이렇게 본다면 무조건 세션 방식이 좋아 보이지만 세션 방식은 서버 부하와 확장성에 있어 단점을 가지고 있다. 인증정보를 서버에서 관리한다는 것은 그만큼 서버에 데이터 접근이 많아진다는 것이고, 이는 서버 부하의 증가를 의미한다.
확장성에 대해선 세션 인증방식을 사용하면 세션 정보는 서버의 메모리상에 존재한다. 이는 서버가 1대일 때는 문제가 되지 않지만, 2대 이상의 컴퓨터를 이용해 서버를 구성할 경우 복수의 컴퓨터의 메모리가 일치하지 않아 인증 정보가 유지되지 않는다.
하지만 확장성에 대해선 서버의 메모리에 세션 정보를 저장하지 않고 Redis와 같은 데이터베이스에 세션 정보를 저장하게 되면 복수의 서버를 이용하더라도 하나의 세션 서버를 가지고 있기 때문에 확장성에 대해서 문제가 사라진다. 하지만 서버의 유지비용은 늘어날 것이다.

## express에서 session 사용

express에서 세션로그인을 사용하기 위해선 `express-session`을 설치해야한다.
```shell
$npm install express-session
```

타입스크립트를 사용하고 있다면 타입 또한 설치해야 한다.
```shell
$npm intall @types/express-session
```

설치가 끝나면 index.ts에 session 설정을 추가해준다.

```ts
// index.ts

import express, { Request, Response, NextFunction } from "express";
import cookieParser from "cookie-parser";
import session from "express-session";
import "./@types/express-session";
import "./@types/express";

const startServer = async () => {
	const app = express();

	// 세션 활성화
	app.use(cookieParser("sessionKey"));
	app.use(
	session({
		secret: "sessionKey", // 비밀키 이대로 쓰지 마시오
		resave: false, // 세션 강제 재 저장 여부
		saveUninitialized: false, // 빈 세션값 저장 여부
		cookie: { httpOnly: true, secure: false, domain: "*" },
		})
	);

	// 라우트
	app.use("/", router);

	// 서버 옵션, 실행
	const options = {
		port: Number(env.getEnv("NODE_PORT")) || 4000,
	};
	app.listen(options.port, "0.0.0.0", () =>
		console.log(`server on!!!${options.port}`)
	);

	return app;
};
startServer();
```

세션을 활성화 하는법은 간단하다 쿠키를 읽을 `cookieParser`와 `session`을 미들웨어로 선언해주면 된다. `session`의 `secret`은 `cookieParser`와 일치해야 한다. 기본 설정은 여기서 끝났다 설정 자체는 정말 간단하다.

세션에 정보를 입력하기 위해선 `req.session`에 정보를 추가해야한다.
컨트롤러로 가서 정보를 추가하기 전 세션인 `req.session`을 콘솔로 출력하면 세션에 대한 정보가 보이게 된다.

```ts
export class AuthController {
	public async signIn(req: Request, res: Response, next: NextFunction) {
		const bodySchema = Joi.object({
			id: Joi.string()
				.regex(/^[a-z0-9]{6,13}$/)
				.required(),
			password: Joi.string()
				.regex(/^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,16}/)
				.required(),
			});
	
		const body = await bodySchema.validateAsync(req.body);
	
		const userInfo = await this.authService.signIn(body, req.session);
		console.log(req.session)
	
		req.session.loginInfo = userInfo;
	
		res.status(200).json({
			isSuccess: true,
			data: userInfo 
		});
	}
}
```
![](https://velog.velcdn.com/images/kisuk623/post/3ebbf18c-761f-47ad-b9dd-96bbf73da6b4/image.png)


위 코드를 실행시키면 서비스 단에서 가져온 유저 정보가 콘솔에는 보이지 않는다. 이는 `req.session`에 유저 정보를 담기 전 콘솔을 출력했기 때문이다.
다시 한번 같은 API를 호출해보자.
![](https://velog.velcdn.com/images/kisuk623/post/f186a338-9992-4c0d-81ba-048b85af3333/image.png)


같은 로그인 API를 호출하니 이전에 `req.session`에 담은 유저 정보가 추가 되었다. 세션은 이처럼 쿠키를 통해 유저를 판별하고, 서버에 저장된 유저 정보를 획득해 인가 작업을 진행한다.

세션 정보를 메모리에 저장해 사용하고자 하면 여기서 더 설정할 것은 없다.
여기까진 JWT토큰 방식보다 세션 방식이 세팅하기 쉽다고 생각이 들것이다. 하지만 세션 정보를 메모리에 저장한다는 것은 서버가 재부팅될 때마다 모든 유저의 로그인 정보가 초기화되며, 수평적 확장을 할 수 없다. 그렇기에 세션 정보를 저장하는 서버를 구축해 이용해 보도록 하자

## Redis

요즘 토이 프로젝트 용도로 AWS 프리티어를 많이 사용한다. 오늘 사용할 것도 AWS에서 제공하는 ElastiCache를 이용해 Redis서버를 만들어 볼 것이다.

AWS에서 ElastiCache에 들어가 지금 시작 > Redis 클러스터 생성으로 들어와 클러스터 설정에 들어간다.

![](https://velog.velcdn.com/images/kisuk623/post/81fe6cf8-d1d0-453b-b5a7-3bc10a0cef0e/image.png)


그 후 필요한 정보를 입력한 뒤, VPC는 기존 서버가 구동 중인 VPC를 지정해 주면 Redis 기본 설정이 끝난다.

Redis생성에 시간이 걸리니 그동안 보안그룹에서 Redis 포트를 열어주도록 하자 Redis포트는 6379이다.

![](https://velog.velcdn.com/images/kisuk623/post/cb7dfdd4-8167-4802-89c5-dc2329006a65/image.png)


Redis 생성이 끝나면 엔드포인트를 기록해 둬야한다. 구성 엔드포인트 혹은 기본 엔드포인트에 작성된 엔드포인트를 복사해 두자.

![](https://velog.velcdn.com/images/kisuk623/post/e40266a7-1191-4c17-8373-dc3cff7a2481/image.png)


### Redis 세션 연결

세션 정보를 Redis에 저장하려면 `connect-redis` 라이브러리가 필요하다.
```shell
$ npm install connect-redis
```

설정이 종료되면 Redis연결을 위해 몇가지 설정을 해 줘야한다.

```ts
// src/config/redis.ts

import { createClient } from "redis";

const redisClient = createClient({ url: process.env.REDIS_URL });

export { redisClient };
```

```ts
// src/index.ts

import express, { Request, Response, NextFunction } from "express";
import cookieParser from "cookie-parser";
import session from "express-session";
import RedisStore from "connect-redis";
import { redisClient } from "./config/redis";
import "./@types/express-session";
import "./@types/express";

const startServer = async () => {
	const app = express();

	// Redis 연결
	redisClient
		.connect()
		.then((res) => console.log("REDIS 연결"))
		.catch(console.error);

	// Redis 저장소 설정
	let redisStore = new RedisStore({
		client: redisClient,
		prefix: "session:", 
	});

	// 세션 활성화
	app.use(cookieParser("sessionKey"));
	app.use(
	session({
		store: redisStore,
		secret: "sessionKey", // 비밀키 이대로 쓰지 마시오
		resave: false, // 세션 강제 재 저장 여부
		saveUninitialized: false, // 빈 세션값 저장 여부
		cookie: { httpOnly: true, secure: false, domain: "*" },
		})
	);

	// 라우트
	app.use("/", router);

	// 서버 옵션, 실행
	const options = {
		port: Number(env.getEnv("NODE_PORT")) || 4000,
	};
	app.listen(options.port, "0.0.0.0", () =>
		console.log(`server on!!!${options.port}`)
	);

	return app;
};
startServer();
```

기존 세션에서 추가된 건 Redis를 연결한 뒤 미들웨어 `session`에 store에 추가해 주면 이제 세션정보는 모두 설정한 Redis에 저장된다. 
env에 Redis 엔드포인트를 넣는 걸 잊지 말자 AWS에서 저장한 엔드포인트 앞에 redis://을 붙여야 정상 작동한다.

해당 세팅을 모두 끝내고 서버를 실행하면 Redis에 접속이 되지 않는다.
생각해보자, 우리는 Redis에 접속하기 위한 엔드포인트만 알고 있지 보통의 데이터베이스가 가지고 있는 계정과 PW를 알지 못한다.
왜 계정과 PW가 없을까? AWS Elasticache는 VPC내부망으로만 접속할 수 있어 ID와 PW를 제공하지 않는다.
그렇기 때문에 로컬환경에서 테스트를 진행할 때는 로컬에 Redis를 설치해 테스트를 진행해야 한다.

### Redis 로컬 설치

Redis는 brew를 통해 간편히 설치할 수 있다.
```shell
$ brew install redis
```

```shell
$ brew services start redis
```

설치 후 실행 명령어를 통해 redis를 실행시키면 간단히 작동하며 `redis-cli` 명령어를 통해 redis에 접속할 수 있다.

```shell
$ redis-cli
```

설치를 확인했으면 로컬의 Redis 엔드포인트를 `redis://localhost:6379` 로 변경해주자. 
EC2에 존재하는 엔드포인트는 AWS에 존재하는 Redis의 엔드포인트를 유지해야한다.

세션을 통해 로그인을 진행하면 redis에 세션 정보가 쌓이게 되는데 redis 상에서 `keys *`을 입력하면 현재 저장중인 데이터를 확인할 수 있다.
Redis는 NoSQL데이터베이스로 자세한건 다음에 더 알아보도록 하고 오늘은 세션에 대해서만 언급하겠다.

![](https://velog.velcdn.com/images/kisuk623/post/e1f4564a-7982-4204-aa79-2da69e45baf3/image.png)


Redis에서 `keys *`명령어를 입력하니 현재 접속중인 세션이 3개가 있다고 출력되어 저장이 잘 된것을 볼 수있다.

`get`명령어를 통해 세션을 조회하면 해당 세션의 값을 볼 수 있다.

![](https://velog.velcdn.com/images/kisuk623/post/f4996182-fe78-4d0d-a71d-946ef47d675a/image.png)


### Redis EC2에서 접속

ElastiCache로 생성한 Redis에 접속하기 위해선 Redis를 생성할때 설정한 VPC에 존재하는 인스턴스를 통해 접속할 수 있다.
해당 인스턴스에 접속한 후 Redis접속 프로그램을 설치해주자

#### 설치

```shell
$ wget http://download.redis.io/redis-stable.tar.gz && tar xvzf redis-stable.tar.gz && cd redis-stable && make
```

```shell
$ sudo cp src/redis-cli /usr/bin
```

#### 접속

```shell
$ redis-cli -h [엔드포인트]
```

설치에 시간이  오래 걸리며, 설치가 끝난 뒤 접속해 보면 로컬에서 접속한 것과 같은 Redis화면을 볼 수 있게 된다. 인스턴스에 작동 중인 프로젝트에 세션 로그인을 하면 ElastiCache의 Redis에도 세션 정보가 추가된 것을 확인할 수 있다.

### 원격 로그아웃

세션 인증 방식의 특징 중 하나가 서버에서 유저인증 정보를 관리한다는 점이었다. 해당 특징을 이용하면 Redis에 저장된 세션 정보를 지워 특정 유저의 인가 작업을 중지할 수 있다.

```ts
redisClient.del(`session:${sessionId}`)
```

sessionId는 컨트롤러의 `req.session.id` 에서 얻을 수 있다. 세션 인증을 진행할 때 데이터베이스에 세션ID를 같이 저장해 준 뒤, 특정 유저의 원격 로그아웃을 하고 싶은 경우 데이터베이스에 저장된 세션ID를 Redis상에서 삭제하면 해당 유저는 서버에 인증/인가 정보가 없기 때문에 사실상 로그아웃된 상태라고 볼 수 있다.


## 겪은 이슈

1. Redis에 세션id가 무한히 증식했다.
	- Redis를 세팅하고 1주일 정도 지난 뒤 ElastiCache의 Redis를 들여다봤더니 세션 정보가 17만 개가 있었다. 아직 개발 중인 단계라 사용자가 들어올 리도 없었고, 1계정당 1개의 세션만 가질 수 있게 설정을 해둬 말이 되지 않았다.
	  이유를 알아보니 session 설정 중 `saveUninitialized` 가 `true`로 되어있으면 정보가 담겨있지 않은 세션조차 저장해 버린다는 것이었다.
	  현재 서버에 ELB를 붙여놔 2초 간격으로 헬스체크를 하는데, 이 헬스체크를 할 때마다 세션이 생성되었던 것이었다. 이 이슈는 `saveUninitialized`를 `false`로 바꾸어 해결했다.
2. 클라이언트에서 세션 정보를 보내지 못함
	- 이는 간단히 해결했는데 클라이언트에서 axios로 통신할 때 `withCredential` 옵션을 `true`로 변경해 줘야 세션ID가 담긴 쿠키를 서버로 보내게 된다.
3. 클라이언트에서 로컬에서 작업할 때 배포된 서버의 쿠키를 받아오지 못함
	- 브라우저에서 요청 헤더에 쿠키를 담아 보내기 위해선 쿠키에 지정된 도메인과 서버가 일치해야 한다. 클라이언트, 서버 둘 다 배포환경일 때는 문제가 없으나 클라이언트는 로컬환경에서 작업할 경우가 많은데 로컬 작업에선 쿠키가 저장되더라도 서버에 API를 전송할 때 헤더에 담기지 않았다.
	  임시 해결법으로 프론트개발자분의 로컬 컴퓨터에 백엔드 서버를 실행시켜서 작업을 진행하도록 했다. 좀 더 나은 방법이 있을거 같지만 여러 가지 시도가 다 실패해 앞으로 개선 방법을 찾아 보려고 함


## 마치며

오늘은 세션 인증 방식에 대해 알아봤다. 요즘엔 인증/인가 방식 하면 다들 JWT를 이용한 토큰 방식만 가르쳐 주는 거 같다. 실제로 나도 JWT방식만 할 줄 알았고, 세션은 그냥 이런 게 있다라고 듣기만 했다.
JWT뿐만아니라 세션 방식도 다들 한 번쯤 사용해 보면 좋지 않을까
확실히 JWT가 서버에 부하도 적고 한번 구축해 두면 기타 신경 쓸 일이 적다는 점에서 장점이 있는 거 같지만, 세션은 보안이 더 유리하다는 점 그리고 서버에서 유저를 관리할 수 있다는 점에서 재미있는 것 같다.

