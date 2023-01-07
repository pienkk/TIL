>Nest.js에서 컨트롤러는 클라이언트에서 들어오는 요청을 처리하고, 클라이언트에 맞는 값을 전달하는 역할을 한다.

## Controller

```ts
@Controller('community')
export class CommunityController {
	constructor(private readonly communityService: CommunityService) {}

	@Get()
	getPosts(@Query() pageNation: QueryDto): Promise<PostListDto> {
		return this.communityService.getPosts(pageNation);
	}

	@Get(':postId')
	getPostDetail(@Param('postId') postId: number): Promise<PostDetailDto> {
		return this.communityService.getPostDetail(postId);
	}

	@Post()
	createPost(@Body() createPostDto: CreatePostDto): Promise<Posts> {
		return this.communityService.createPost(createPostDto);
	}

	@Patch(':postId')
	updatePost(@Param('postId') postId: number,
	@Body() updatePostDto: UpdatePostDto): Promise<void> {
		return this.communityService.updatePost(postId, updatePostDto);
	}

	@Delete(':postId')
	removePost(@Param('postId') postId: number): Promise<void> {
		return this.communityService.removePost(postId);
	}
}
```

컨트롤러의 기본적인 CRUD 형태이다. Express를 기준으로 보면 라우트와 컨트롤러를 합쳐놓은 듯한 모습을 볼 수 있다.
기능별 인스턴스 상단에 REST 메서드가 적혀있는 데코레이터를 볼 수 있는데, 해당 메서드와 엔드포인트가 입력 될 경우 해당 인스턴스가 실행된다.

`GET localhost:3000/community` 을 요청 할 경우 `GetPosts` 인스턴스가 실행되고 `GET localhost:3000/community/1` 을 요청 할 경우 `GetPostDetail` 인스턴스가 실행된다.

Express서버의 경우 클라이언트에서 전달한 값은 모두 `req`에 담기고 `req.body` 와 같이 객체에 접근해 원하는 값을 사용 했다. Nest에서도 해당 방법을 통해 요청값에 접근 할 수 있지만, 권장하지 않는다.

| Decorator | Express     | 설명              |
| --------- | ----------- | ----------------- |
| @Req()    | req         | 요청 값           |
| @Res()    | res         | 반환 값           |
| @Param()  | req.params  | param 파라미터 값 |
| @Query()  | req.query   | query 파라미터 값 |
| @Body()   | req.body    | Body 요청 값      |
| @Headers  | req.headers | Header 요청 값    |

필요한 데코레이터를 사용 한 뒤 인자로 요청 값을 추가하고, 해당 데코레이터 우측에 요청값과 타입을 지정해 주면 해당 매개 변수에 요청 값이 할당 된다.
`@Get(':postId')` 와 `@Param('postId')` 을 사용한 경우, `GET localhost:3000/community/3` 요청 했을 때 postId의 값으로 3이 할당 된다.
쿼리나 바디의 경우 필요한 요청 값이 보통 둘 이상이다. 여러개의 값을 요청 받을 경우, 값 마다 데코레이터를 계속 입력해 줘야 한다. 이는 가독성도 좋지 않고, 해당 값에 내가 원하지 않는 값이 들어오는 경우가 생긴다.
이 때 사용하는 것이 DTO이다. DTO는 요청 값을 유효성을 점검하고 검증하는 역할을 한다.

## DTO (Data Transfer Object)

>DTO(Data Transfer Object)는 데이터의 유효성을 검증하기 위해 사용한다. 내가 설정하지 않은 값이 들어오거나, 설정하지 않은 타입의 값이 들어올 경우, 에러를 발생 시킨다.

유효성 검증을 위해 `class-validator` 와 `class-transformer` 를 설치 해 준다.
그 후 `main.ts`를 아래와 같이 수정해 준다.

```ts
async function bootstrap() {
	const app = await NestFactory.create(AppModule);
	app.useGlobalPipes( new ValidationPipe({
		whitelist: true, // DTO에 작성한 값만 수신
		forbidNonWhitelisted: true, // DTO에 작성된 필수값이 수신되지 않을 경우 에러
		transform: true, // DTO의 타입을 변환
		}),
	);
	await app.listen(3000);
}
bootstrap();
```

### Pipe
>Nest에서 Pipe는 데이터 타입의 변환 혹은 값의 유효성 검증을 해주는 역할을 한다. 


![](https://velog.velcdn.com/images/kisuk623/post/c8362693-e826-49fc-ab23-3d4c9386ffd1/image.png)


Pipe는 요청값이 컨트롤러로 도달하기 전 지나가는 곳이다. Pipe의 종류는 9가지 있으며 오늘 사용할 Pipe는 유효성 검증 Pipe다.
`app.useGlobalPipes` 는 Express로 비유하자면 상시 작동하는 미들웨어다. 모든 요청값은 해당 Pipe를 지나야 컨트롤러에 도달 할 수 있다.
`ValidationPipe` 는 유효성을 검증하는 Pipe다 `class-validator`,  `class-transformer`  사전에 설치한 두개의 패키지를 이용해 요청 값의 유효성을 검증하고, 내가 원하는 경우 타입의 변경을 시켜준다.

`@Param()` 이나 `@Query()` 데코레이터를 사용해 number 값을 받을 경우, 해당 값은 string값으로 할당 된다. 이럴 때 `transform`을 true 로 설정 한 뒤, 컨트롤러에서 변환을 원하는 값의 타입을 number로 지정하면 해당 값은 number로 들어오게 된다.

### DTO 예제
```ts
export class CreatePostDto {
	@IsString()
	@IsNotEmpty()
	private readonly title: string;

	@IsString()
	@IsNotEmpty()
	private readonly description: string;
}

export class QueryDto {
	@IsNumber()
	@IsOptional()
	private readonly page: number = 1;

	@IsNumber()
	@IsOptional()
	private readonly number: number = 10;

	@IsString()
	@IsOptional()
	private readonly title: string = '';
}
```

`@IsString()` , `@IsNotEmpty()` 와 같은 데코레이터를 사용해 유효성을 점검 할 수 있다. `@IsNotEmpty()`은 필수 값인 경우, `@IsOptional()`은 옵션 값인 경우, 옵션 값은 값이 할당되지 않았을 때 기본 값을 지정해 줄 수도 있다.

#### 자주 사용하는 검증 데코레이터

| Decorator     | 설명                    |
| ------------- | ----------------------- |
| @IsString()   | 문자열 검증             |
| @IsInt()      | Int값 검증              |
| @IsBoolean()  | Boolean값 검증          |
| @IsEmail()    | 이메일 형식 검증        |
| @IsArray()    | 배열 값 검증            |
| @IsEnum()     | Enum값 검증             |
| @IsNumber()   | 숫자값 검증             |
| @IsDate()     | 날짜 값 검증            |
| @IsOptional() | 해당 값을 옵션으로 할당 |
| @MaxLength()  | 최대 길이 제한          |
| @MinLength()  | 최소 길이 제한          |
| @Length()     | 길이 제한               |
| @Matches()    | 정규표현식 검증        |
| @Min()        | 최솟값                  |
| @Max()        | 최댓값                        |

이 외에도 수 많은 데코레이터가 존재하니 [공식문서](https://github.com/typestack/class-validator)를 참고 하자!

## 의존성 주입(Dependency Injection)

### 의존(Dependency)란?
> A의 의존대상 B가 변하면 A가 영향을 받는다.
> 하나의 인스턴스가 다른 인스턴스를 의존할 경우, 인스턴스의 이름이 변경되거나 기능이 변경될 경우, 의존대상의 코드를 수정해야 하는 경우가 생긴다.

### 의존성 주입이란?
>클래스 외부에서 객체를 생성하여 해당 객체를 클래스 내부에 주입하는 것이다.
>의존성 주입은 클래스간의 결합도를 느슨하게 만들어 준다. 결합도가 느슨하다는 것은 하나의 클래스가 변경 되었을 때 연결되어 있는 다른 클래스를 변경할 필요성이 적어진다는 뜻이다.

### 의존성 주입의 장점

#### 의존성이 줄어든다
의존하고 있다는것은 의존대상이 변화에 따라 현재의 코드를 수정해야 한다는 것이다. 의존성 주입을 사용하면 현재의 코드를 수정하지 않아도 되거나, 수정할 코드가 줄어들게 된다.

#### 재사용의 용이성
기능을 별도로 구분하여 구현하기 때문에 인스턴스의 재사용이 원할하다.

#### 유닛 테스트
프로젝트의 규모가 커지면 테스트코드는 필수적이다. 의존성 주입을 위해 분리한 기능별로 테스트가 용이하다.

#### 가독성
분리된 기능들을 재사용해서 사용해 전체적인 가독성이 증가되고 흐름을 보기에 용이 해진다.

### 코드 예제

```ts
export class CommunityController {
	constructor(private readonly communityService: CommunityService) {}

	@Get()
	getPosts(@Query() pageNation: QueryDto): Promise<PostListDto> {
		return this.communityService.getPosts(pageNation);
	}
}

@Injectable()
export class CommunityService {
	getPosts(pageNation: QueryDto):{ .... }
}
```

`constructor(private readonly communityService: CommunityService) {}` 해당 코드가 현재 컨트롤러단에서 서비스단의 의존성을 주입하는 코드다.
서비스단을 생성자함수로 생성해 컨트롤러단에 주입 함으로써 컨트롤러단에서 서비스단의 인스턴스를 불러와 실행 시킬 수 있는 것이다.
의존성을 주입하고자 하는 원본 클래스에는 `@Injectable()` 데코레이터가 필요하다.


## 요약

Nest의 컨트롤러는 Express의 라우트와 컨트롤러를 합쳐놓은 모습이고, 각종 데코레이터를 이용해 값을 전달 받거나 Http 메서드를 조절 가능하다.

DTO와 Pipe를 사용하면 입력값의 유효성 검증, 기본값, 타입 변경등의 결과를 얻을 수 있다.

Nest는 강력한 의존성 주입을 통해 이뤄져 있다. 의존성 주입의 개념을 확실히 하고 넘어가자