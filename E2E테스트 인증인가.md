>백엔드 API는 보통 인증되지 않은 유저의 접근을 차단하고, 인증된 유저의 구별을 위해 JWT나 Session을 이용한 인증 인가 방식을 주로 사용한다.
>UnitTest에서는 각각의 로직을 테스트 하기 때문에 인증/인가에 대해 크게 신경쓰지 않고 테스트가 가능하지만 E2E테스트는 API의 시작부터 끝까지 모두 테스트를 하기 때문에 인증/인가 또한 받아야 작동이 가능 하다.
>모든 테스트 환경은 독립적으로 작동이 되어야 하고, 각각의 테스트가 서로 영향을 끼치면 안된다.
>오늘은 E2E테스트에서 인증/인가를 하는법을 알아 보고자 한다.

## 인증/인가

이번 테스트 환경에서 사용할 인증/인가 방식으로 나는 Session로그인 방식을 `express-session` 라이브러리를 이용해 테스트를 진행할 것이다.

### Session로그인

express-session는 유저의 로그인을 했을때 유저 정보를 메모리에 저장하며, 식별할수 있는 쿠키를 클라이언트에게 전달한다.
클라이언트는 그 뒤 API요청을 할때마다 쿠키를 헤더에 실어서 보내면 서버는 해당 쿠키를 통해 유저의 로그인 정보를 해석해 인가 작업을 하게 된다.

```ts //usercontroller.ts
public async signIn(req: Request, res: Response, next: NextFunction) {
	try {
		const bodySchema = Joi.object({
			id: Joi.string()
				.regex(/^[a-z0-9]{6,13}$/)
				.required(),
			password: Joi.string()
				.regex(/^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,16}/)
				.required(),
		});

		const body = await bodySchema.validateAsync(req.body);

		const userInfo = await this.authService.signIn(body, req.ip, req.session);

		req.session.loginInfo = userInfo;

		res.status(200).json({ data: userInfo });
	} catch (error) {
		console.error(error);
		next(error);
	}
}
```

유저가 로그인에 성공하면, 세션에 해당 로그인 정보를 받고, 클라이언트에도 해당 로그인 정보를 반환한다. 클라이언트는 해당 정보를 세션/로컬 스토리지에 저장해 유저정보를 파악하고, 헤더에 담긴 쿠키를 이용해 API서버와의 인증/인가 작업을 진행하게 된다.

### Test

여기서 API서버는 헤더에 담긴 쿠키를 이용해 사용자를 판별한다. 하지만 테스트는 독립적으로 시행되어 반환받은 쿠키를 헤더에 실어서 요청을 하지 못한다.
그렇기에 우리는 인가가 필요한 API를 실행하기 전에 인증을 받아 그 후에 실행될 API의 헤더에 쿠키를 실어서 같이 보내면 인가처리가 완료된 Test가 완료 된다.

```ts
describe("Post E2E 테스트", () => {
	let cookie: any;
	beforeAll((done) => {
		// 유저 로그인, 세션 발급
		request(app)
			.post("/user/signin")
			.send({
				id: "gisuk623",
				password: "Qwer12!232",
			})
			.end((err, res) => {
				cookie = res.headers["set-cookie"].pop().split(";")[0];
				done();
			});
	});

	describe("POST /post 게시글 생성", () => {
	// 성공
	it("게시글 생성 성공 후 게시글 내용을 반환 받는다", async () => {
		const result = await request(app)
		.post("/post")
		.set("Cookie", cookie)
		.send({
			name: "게시글 테스트",
			description: "게시글 내용"
		});

		expect(result.status).toBe(201);
		expect(result.body.data.postIdx).toBeDefined();
		expect(result.body.data.name).toBe("게시글 테스트");
		expect(result.body.data.description).toBe("게시글 내용");
	});
});
```

게시글 생성 테스트를 진행하기 전, `beforeAll` 을 통해 로그인은 먼저 진행해 주었다. 로그인을 진행한 후 반환된 헤더의 쿠키를 `cookie`변수에 저장한 뒤 테스트를 할 때 해당 쿠키를 `set`을 통해 헤더에 같이 실어서 보내주면 인증 인가에 대한 테스트가 끝나게 된다.

### 유저의 권한이 여러개 일 경우

프로젝트를 진행하다 보면 유저마다 권한이 다를경우가 있다. 이럴 때는 `describe`를 유저의 권한마다 나눠 실행하게 되면 각각의 블록은 해당 권한을 가진 유저에 대한 테스트를 구축하는데 용이하다.

```ts
describe("Post E2E 테스트", () => {
	describe("1번째 유저", () => {
		let cookie: any;
		beforeAll((done) => {
			// 유저 로그인, 세션 발급
			request(app)
				.post("/user/signin")
				.send({
					id: "gisuk623",
					password: "Qwer12!232",
				})
				.end((err, res) => {
					cookie = res.headers["set-cookie"].pop().split(";")[0];
					done();
				});
		});
	
		describe("PATCH /post 게시글 수정", () => {
		// 성공
		it("게시글 수정 성공 후 게시글 내용을 반환 받는다", async () => {
			const result = await request(app)
			.post("/post/75b0266f-4803-4b77-ab6b-380105fc4aa2")
			.set("Cookie", cookie)
			.send({
				name: "게시글 테스트",
				description: "게시글 내용"
			});
	
			expect(result.status).toBe(200);
			expect(result.body.data.postIdx).toBeDefined();
			expect(result.body.data.name).toBe("게시글 테스트");
			expect(result.body.data.description).toBe("게시글 내용");
		});
	});
	
	describe("2번째 유저", () => {
		let cookie: any;
		beforeAll((done) => {
			// 유저 로그인, 세션 발급
			request(app)
				.post("/user/signin")
				.send({
					id: "gisuk333",
					password: "Qwer12!qw2",
				})
				.end((err, res) => {
					cookie = res.headers["set-cookie"].pop().split(";")[0];
					done();
				});
		});
	
		describe("PATCH /post 게시글 수정", () => {
		// 실패
		it("타인의 게시글을 수정 시도할 경우, 타인의 게시글을 수정할수 없다는 메시지를 받는다.", async () => {
			const result = await request(app)
			.post("/post/75b0266f-4803-4b77-ab6b-380105fc4aa2")
			.set("Cookie", cookie)
			.send({
				name: "게시글 테스트",
				description: "게시글 내용"
			});
	
			expect(result.status).toBe(400);
			expect(result.body.data.message).toBe("타인의 게시글을 수정할 수 없습니다.")
		});
	});
});
```

위 처럼 두개의 계정을 이용해 테스트 또한 가능하다.

## 마치며

>오늘은 E2E테스트에서 중요한 인증/인가 방식에 대해 알아보았다. 인증/인가는 API를 구성함에 있어서 가장 중요한 부분이라고 생각된다. 인증/인가가 제대로 되지 않는다면 누구나 나의 API에 접근할 수 있다는 것이고, 그것은 데이터베이스가 오염이 될 수 도 있다는 것이다.
>인증/인가 부분을 mocking하여 진행할 수도 있지만, 최대한 실제와 같은 환경을 구성해 테스트를 진행하는것이 옳다고 생각해 mocking이 아닌 실제 인증/인가 방식을 통한 E2E테스트를 진행했다.
>Redis를 통해 세션을 구축해 테스트를 함에 있어 우여곡절이 많았으나 좋은경험을 했다.