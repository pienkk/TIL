>이전 Nest.js에서 Jest를 이용한 Unit Test를 진행했다. 비즈니스로직을 검증하는 Unit Test도 물론 중요하지만, 백엔드 API를 호출하는 일련의 과정을 테스트 하는 E2E(End To End)가 더욱 중요하다고 생각한다.
>E2E테스트는 Unit Test와 다르게 소셜로그인과 같은 외부 API를 의존하는 경우를 제외하고는 Mocking을 진행하지 않는다. Mocking을 이용해 테스트를 진행하면 내가 만든 함수가 제대로 작동되는지 보증할 수 없기 때문이다.

## 테스트환경과 서버 실행환경 분리

Unit Test는 Repository를 Mocking해 데이터베이스와 직접적으로 연결을 하지 않고 테스트를 진행했다. 하지만 E2E테스트는 전체적인 API의 Flow를 테스트 해야하기 때문에 데이터베이스와 직접 연결해 테스트를 진행한다.
하지만, 실제 사용하는 데이터베이스를 사용할 경우, 테스트에 의해 데이터가 소실될 수 있다. 이 때문에 테스트환경과 실제 서버실행환경을 분리시켜줄 필요가 있다.

```json
// package.json

"scripts": {
	"build": "tsc",
	"start": "pm2 dist/src/server.js",
	"dev": "nodemon src/server.ts",
	"test:e2e": "NODE_ENV=test jest --config jest-e2e.config.js"
},
```

```ts
// src/app.ts

const createApp = () => {
	const app = express();

// TypeORM DB 연결
	if (process.env.NODE_ENV !== "test") {
		dataSource .initialize()
		.then(() => {
			console.log("Database initialize!!");
		})
		.catch((err) => {
			console.error(err);
		});
	}

/** 
	기타 미들웨어
**/

	return app;
};

export { createApp };
```

```ts
// src/config/typeormConfig.ts

import { join } from "path";
import { DataSource } from "typeorm";

require("dotenv").config();

const databaseName =
process.env.NODE_ENV === "test" ? process.env.DB_TESTNAME : process.env.DB_NAME;

// TypeORM Database 설정
const dataSource = new DataSource({
	type: "mysql",
	host: process.env.DB_HOST
	port: Number(process.env.DB_PORT)
	database: databaseName,
	username: process.env.DB_USER
	password: process.env.DB_PASSWORD,
	entities: [join(__dirname, "../**/*Entity.{ts,js}")],
	synchronize: process.env.NODE_ENV === "test" ? true : false,
	logging: process.env.NODE_ENV === "test" ? false : true,
	dropSchema: process.env.NODE_ENV === "test" ? true : false,
});

export { dataSource };
```

package.json 에서 `NODE_ENV`를 test로 지정해 주고 기존 `NODE_ENV`가 test일 경우 app.ts에서는 데이터베이스 연결을 하지 않는다. 이는 E2E 환경에서 데이터베이스의 중복 연결을 막기 위해서이다.
그 후 ormconfig에서 test환경을 분리 시켜 준다. `NODE_ENV`가 test일 때 접속하는 데이터베이스명을 분리하고, 각종 옵션들을 test환경과 분리 시켜준다.
`synchronize` 옵션은 Entity파일을 읽고 자동으로 데이터베이스의 테이블을 만들어 주고, `dropSchema` 옵션은 해당 옵션이 true일 때 데이터 베이스 연결 전 테이블을 모두 지워준다. 이 덕에 테스트 환경마다 동일한 결과를 얻을 수 있다.

### supertest 라이브러리를 이용한 E2E 테스트

```ts
// src/test/app.e2e.spec.ts

import { dataSource } from "../config/typeormConfig";
import axios from "axios";
import request from "supertest";
import { Express } from "express-serve-static-core";
import { createApp } from "../app";
import { KakaoInfo, ResponseKaKaoToken } from "../types/KakaoInfo";
import { typeormSeed } from "./seeds";

let app: Express;
jest.mock("axios");

beforeAll(async () => {
	app = createApp();
	await dataSource.initialize();
	typeormSeed();
});

afterAll(async () => {
	await dataSource.destroy();
});

describe("user E2E", () => {
	// 성공
	it("소셜 로그인 성공 시 jwt, nickname, profileImage를 반환한다.", async () => {
		const KakaoAccessToken: ResponseKaKaoToken = {
			access_token: "accesstoken",
			token_type: "access_token",
			refresh_token: "RefreshTopken",
			scope: "profile_image, nickname",
		};

		const KakaoUserInfo: KakaoInfo = {
			id: 1234567,
			kakao_account: {
				email: "kisuk623@gmail.com",
				profile: {
					nickname: "기석",
					profile_image_url: "http://naver.com",
				},
			},
		};

		// 카카오 api mocking
		const axiosPostSpy = jest
		.spyOn(axios, "post")
		.mockResolvedValue({ data: KakaoAccessToken });
		const axiosGetSpy = jest
		.spyOn(axios, "get")
		.mockResolvedValue({ data: KakaoUserInfo });
	
		const result = await request(app).get("/user/auth").query({ code: "kakaocode" });

		expect(result.status).toBe(200);
		expect(result.body.nickname).toBe("기석");
		expect(result.body.profileImage).toBe("http://naver.com");
	});
});
```

```ts
// src/services/UserService.ts

import { UserRepository } from "../repositories/UserRepository";
import { ResponseSignInDto, SignInDto } from "../dtos/SignInDto";
import { AuthService } from "./AuthService";
import { KakaoInfo } from "../types/KakaoInfo";

export class UserService {
	private readonly userRepository = new UserRepository();
	private readonly authService = new AuthService();

	/**
	* 소셜 로그인 / 회원가입
	*/
	public async signIn({ code }: SignInDto): Promise<ResponseSignInDto> {
		// 카카오 소셜 로그인 / 유저 정보 획득
		const userInfo: KakaoInfo = await this.authService.getKakaoUserInfo(code);

		// 회원가입 여부 확인
		let user = await this.userRepository.findOne({
			where: { email: userInfo.kakao_account.email },
		});

		// 회원가입되지 않은 유저의 경우
		if (!user) {
		// 회원 가입 후 유저 정보 반환
			user = await this.userRepository.save({
				social_id: userInfo.id,
				email: userInfo.kakao_account.email,
				nickname: userInfo.kakao_account.profile.nickname,
				profile_image: userInfo.kakao_account.profile.profile_image_url,
			});
		}

		// Jwt 발급
		const accessToken = this.authService.createJwt(user.id);

		return {
			accessToken,
			nickname: user.nickname,
			profileImage: user.profile_image,
		};
	}
}
```

기존 Unit Test에선 해당 함수의 로직에 필요하지 않은 것들은 모두 Mocking처리 했지만, E2E테스트에서는 외부 API에 대한 부분만 Mocking처리 했다.
외부 API는 호출할 때 마다 값이 바뀌기 때문에 Test는 실행될 때마다 동일 해야 한다는 원칙에 위배 되어, Mocking을 진행 했다.

테스트는 supertest라이브러리를 사용해 진행하면 된다. request메소드에 `src/app.ts` 의 app을 할당한 뒤, API의 메소드와 엔드포인트, 그리고 전달할 값을 적어주면 해당 엔드포인트로 해당 값을 이용해 테스트가 진행 된다.

![](https://velog.velcdn.com/images/kisuk623/post/7ef42970-eabc-4e2b-82e5-2e044a26e971/image.png)


## Unit Test와 E2E Test 환경 분리

Jest의 테스트는 병렬로 처리 되어, 테스트 환경간에 간섭을 줄 수도 있고, 필요에 따라 Unit Test나 E2E 테스트만 진행하고 싶을 때도 있다.
이를 위해 Unit Test와 E2E Test 환경을 분리시켜 보자.

![](https://velog.velcdn.com/images/kisuk623/post/ebf54a7f-98fe-4b64-8a38-3fe368e8a809/image.png)


```js
// jest-unit.config.js

module.exports = {
	moduleFileExtensions: ["ts", "tsx", "js", "json"],
	rootDir: ".",
	testRegex: ".*\\.test\\.ts$",
	transform: {
		"^.+\\.(t|j)s$": "ts-jest",
	},
	collectCoverageFrom: ["**/*.(t|j)s"],
	coverageDirectory: "../coverage",
	testEnvironment: "node",
};

//-------------------------

// jest-e2e.config.js

module.exports = {
	testRegex: ".*\\.spec\\.ts$",
	// 이외는 동일
}
```

```json
// package.json

"scripts": {
	"build": "tsc",
	"start": "pm2 dist/src/server.js",
	"dev": "nodemon src/server.ts",
	"test:unit": "NODE_ENV=test jest --config jest-unit.config.js",
	"test:e2e": "NODE_ENV=test jest --config jest-e2e.config.js"
},
```

jest.config 파일을 두가지 만들어 각각 실행시키는 파일을 분리 시켜 주고, package.json에서 테스트 스크립트를 둘로 나누어 Unit Test와 E2E Test으로 분리 시켜주었다.

![](https://velog.velcdn.com/images/kisuk623/post/dee7d807-c0e7-460e-9cbb-2e11b8112fb6/image.png)


테스트 환경이 분리되어 실행되는 모습


## 마치며

오늘은 백엔드에서 중요하다고 생각되는 E2E Test를 알아 보았다. 기존에는 Unit Test로만 테스트 코드를 작성했으나, Unit Test는 함수들의 로직만 테스트를 하는것이라 조금 부족한 감이 있었다.
이번에 E2E Test와 Unit Test를 같이 진행하며, 전체적인 Flow와 각자의 함수들을 테스트 할 수 있게되어 더이상 API를 호출하는 식의 테스트가 아닌 테스트 코드 실행만으로 테스트를 할 수 있다는 점에서 흥미롭고, 배포 자동화에 한발 더 나아간 느낌이 든다.
앞으로는 스트레스테스트와 같이 서버가 실행되는 환경에서의 테스트 또한 진행해보고 싶다.