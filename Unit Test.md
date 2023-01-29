>테스트 코드는 프로그래밍에 있어 매우 중요하며, 큰 비중을 차지하고 있다.
>테스트 코드에는 크게 유닛 테스트, 기능 테스트, 통합 테스트 3가지로 나뉜다. 오늘은 테스트 단위 중에 가장 작은 유닛 테스트를 알아보고자 한다.
>테스트 코드를 짜기 쉬운 코드는 좋은 코드이다. 라는 말도 있다. 테스트 코드를 짜기 쉽다는 것은 그만큼 구조적으로 잘 짜인 코드라는 뜻이다.
>테스트 코드를 작성함으로 버그를 줄이고 신뢰성이 증가한다. 리팩토링을하더라도 테스트 코드를 작동시켜 기존과 같은 동작을 하는지 알 수 있고, 코드 작성자의 의도 또한 알 수 있다.

## Jest

>Jest는 코드가 동작하는지 Test Case를 통해 확인하기 위한 테스팅 프레임워크 이다.

Jest는 Nest.js에 기본적으로 탑재되어 있다. Api를 만들면 해당 폴더 안에`.spec.ts` 가 붙은 파일이 같이 생성되는데 해당 파일이 테스트 코드를 작성하기 위한 파일이다.

```ts
// app.controller.spec.ts

import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { AppService } from './app.service';

describe('AppController', () => {
	let appController: AppController;

	beforeEach(async () => {
		const app: TestingModule = await Test.createTestingModule({
		controllers: [AppController],
		providers: [AppService],
		}).compile();
	appController = app.get<AppController>(AppController);
	});

	describe('root', () => {
		it('should return "Hello World!"', () => {
			expect(appController.getHello()).toBe('Hello World!');
		});
	});
});
```

### Jest 기초 문법

`describe()`는 테스트의 단위를 묶어 주며, 묶은 단위의 설명을 기재해 준다. `describe()` 안에 `describe()`를 작성할 수도 있다.
`beforeEach()`와 `afterEach()`는 각각의 Test Case를 실행하기 전, 후에 작동하는 코드이다. Test Case가 3개가 있으면 `beforeEach()`와 `afterEach()`는 각각 3번씩 실행될 것이다.
조금 다른 역할을 하는 함수로 `beforeAll()`과 `afterAll()`이 있다. 두 함수는 해당 파일에서 테스트가 시작하기 전 한 번, 모든 테스트가 끝난 뒤 한번 실행되는 함수이다. Test Case마다 작동이 필요한 코드가 아닌 경우에는 `beforeAll()`과 `afterAll()`을 사용하는 게 효율적이다.
`it()`는 `test()`와 동일하게 작동하며, 둘 중 어떤 함수를 사용해도 상관없다. `describe()`로 묶은 단위에 테스트를 실행하는 함수이다.
`expect()`는 Test Case의 실행 결과가 내가 원하는 값과 일치하는지 확인하는 함수이다. `toBe()` 메서드는 평문을 비교할 때 사용하고, `toEqual()`은 객체와 같은 값을 비교할 때 사용한다.

### Mocking

>Mocking은 프론트엔드 개발자라면 익숙한 개념이지만, 신입 백엔드개발자들 중에서는 처음 접하는 개념일 수도 있다. Mock의 사전적 의미는 '가짜'이다 Mocking은 함수의 실행 결괏값을 내가 지정한 값으로 대체하는 작업을 의미한다.
>백엔드의 비즈니스로직은 데이터베이스와 깊이 연관되어 있다. 그렇기에 Service 레이어를 테스트를 하기 위해선 데이터베이스와 연결을 해야 하는데 테스트하는 과정에서 문제점이 발생할 수 있다. 하지만 Mocking을 사용하면, Service 레이어에서 데이터베이스에 연결하지 않고, 개발자가 지정해준 값을 바탕으로 테스트를 진행하게 되어, 좀 더 독립적인 Test Case가 완성된다.

#### jest.fn()과 jest.spyOn()

Jest에는 가짜함수를 새로 생성하는 `jest.fn()`과 기존 함수의 작동은 유지하되 해당 함수의 결괏값을 가짜로 대체해주는 `jest.spyOn()`이 있다.
두 가지 다 가짜값을 반환하고자 하는 목적은 같지만, `jest.spyOn()`은 가짜 값을 생성하지 않으며, 해당 함수가 특정인자로 호출되었는지 알아내는 등의 작업도 가능하다.

```ts
// user.service.spec.ts
const mockUserRepository = () => ({
	findOneBy: jest.fn(),
	create: jest.fn(),
	save: jest.fn(),
});
type MockRepository<T = any> = Partial<Record<keyof Repository<T>, jest.Mock>>;

describe('UserService', () => {
	let userService: UserService;
	let jwtService: JwtService;
	let userRepository: MockRepository<UserEntity>;

	beforeAll(async () => {
		const module: TestingModule = await Test.createTestingModule({
			providers: [UserService,
				{
					provide: getRepositoryToken(UserEntity),
					useValue: mockUserRepository(),
				}]}).compile();

		userService = module.get<UserService>(UserService);
		jwtService = module.get<JwtService>(JwtService);
		userRepository = module.get<MockRepository<UserEntity>>(
			getRepositoryToken(UserEntity));
	});

	describe('createUser', () => {
		const createUserInput: CreateUpdateUserDto = {
			name: '기석',
			email: 'kkk@gmail.com',
			password: 'Qwer1234!@',
		};
		const createUser = UserEntity.of({
			...createUserInput
		});
		const userInfo = UserEntity.of({
			id: 1,
			...createUser,
			deleted_at: null,
		});

		it('유저 생성에 성공 시 유저 정보를 반환한다.', async () => {
			const userRepositoryFindOneSpy = jest
			.spyOn(userRepository, 'findOneBy')
			.mockResolvedValue(undefined);
			const userRepositoryCreateSpy = jest
			.spyOn(userRepository, 'create')
			.mockResolvedValue(createUser);
			const userRepositorySaveSpy = jest
			.spyOn(userRepository, 'save')
			.mockResolvedValue(userInfo);

			const result = await userService.createUser(createUserInput);

			expect(result).toEqual(userInfo);
		});

		it('존재하는 유저를 생성 요청 시 유저가 존재하다는 예외를 던진다.', async () => {
			const userRepositoryFindOneSpy = jest
			.spyOn(userRepository, 'findOneBy')
			.mockResolvedValue(userInfo);

			const result = async () => {
				await userService.createUser(createUserInput);
			};

			expect(result).rejects.toThrowError(
				new HttpException('User already exists', HttpStatus.CONFLICT),
			);
		});
	});
});
```

```ts
// user.service.ts
@Injectable()
export class UserService {
	constructor(
		@InjectRepository(UserEntity)
		private readonly userRepository: Repository<UserEntity>) {}
	async createUser(args: CreateUpdateUserDto): Promise<UserEntity> {
		const { email } = args;
		const user = await this.userRepository.findOneBy({ email });

		if (user && !user.deleted_at)
			throw new HttpException('User already exists', HttpStatus.CONFLICT);

		if (user && user.deleted_at) {
			user.deleted_at = null;
			return await this.userRepository.save(user);
		}

		const userEntity = this.userRepository.create(args);
		return await this.userRepository.save(userEntity);
	}
}
```

위 코드는 userService의 회원가입에 대한 Test Case와 Service 레이어에 대한 코드이다.
mockUserRepository를 만들고 Test Case에 사용할 메서드 들을 `jest.fn()`을 이용해 선언해 준다.  그후,모듈에서 해당 Repository를 주입한다.
이제 `createUser` 함수를 실행시키는데 필요한 값, 각 메서드 들의 리턴값과 최종값을 선언해 준뒤, mocking을 통해 각 Repository의 반환 값을 지정해 준다.
반환 값 지정이 끝나면 변수에 함수의 결괏값을 담은 뒤, `.toEqual()`을 이용해 내가 원하는 값과 비교를 한다. 
예외 처리의 경우 `.rejects.toThrowError()`를 이용해 예외값을 비교한다.
예외 처리를 진행할 경우 `expect()`함수 안에서 실행시켜야 한다. `expect()`함수 밖에서 실행될 경우 예외 처리로 진행되어 다음 코드로 진행이 되지 않기 때문에 상단의 예시 코드에는 result를 함수로 만들어 `expect()` 함수 안에서 실행시켜 주었다.

#### jest.fn() 미사용

```ts
describe('UserService', () => {
	let userService: UserService;
	let jwtService: JwtService;
	let userRepository: UserRepository;

	beforeAll(async () => {
		const module: TestingModule = await Test.createTestingModule({
			providers: [UserService,UserRepository]
		}).compile();

		userService = module.get<UserService>(UserService);
		jwtService = module.get<JwtService>(JwtService);
		userRepository = module.get<UserRepository>(UserRepository));
})
```

`jest.fn()`을 사용하면 함수가 실제로 작동하는 게 아닌 가짜 함수만 실행된다. 그렇기에 예상치 못한 에러를 지나치게 될 수도 있다. 이런 경우 `jest.fn()`을 사용하지 않고 직접 Repository의 메서드를 사용하되, `spyOn()`으로 각 함수의 실행 결괏값만 Mocking 해 사용하면 메서드의 필요 조건도 만족하는 Test Case를 작성할 수 있다.
[jest.fn()을 사용한 Test Code](https://github.com/pienkk/maum-coding-test/blob/main/src/user/user.service.spec.ts)
[jest.fn()을  사용하지 않은 Test Code](https://github.com/pienkk/maum-coding-test/blob/main/src/post/post.service.spec.ts)

### 자주 사용하는 Mocking Method

#### mockFn.mockImplementation()

```ts
const mockFn = jest.fn(scalar => 42 + scalar);

mockFn(0); // 42
mockFn(1); // 43
```

결괏값을 Mocking 하는게 아닌 직접 가짜 함수를 만들어 준다.

#### mockFn.mockImplementationOnce()

```ts
const mockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

mockFn((err, val) => console.log(val)); // true
mockFn((err, val) => console.log(val)); // false
```

하나의 함수에 한 번 호출의 결과를 만들어 준다. 위 예시는 2개의 결괏값을 만들어 2번의 호출이 각각 다른 값을 출력했다.

#### mockFn.mockReturnValue()

```ts
const mock = jest.fn();

mock.mockReturnValue(42);
mock(); // 42

mock.mockReturnValue(43);
mock(); // 43
```

동기 함수가 호출될 때 반환 값을 지정해 준다. 

#### mockFn.mockReturnValueOnce()

```ts
const mockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockReturnValueOnce('first call')
  .mockReturnValueOnce('second call');

mockFn(); // 'first call'
mockFn(); // 'second call'
mockFn(); // 'default'
mockFn(); // 'default'
```

동기 함수에 한 번의 호출 당 반환 값을 지정해 준다. `mockReturnValueOnce`를 선언한 횟수 보다 많은 횟수를 호출할 경우 `mockReturnValue` 값을 반환한다.

#### mockFn.mockResolvedValue()

```ts
test('async test', async () => {
  const asyncMock = jest.fn().mockResolvedValue(43);

  await asyncMock(); // 43
});
```

비동기 함수가 호출될 때 반환 값을 지정해 준다. `mockReturnValue`와 다른 점은 동기 함수인가, 비동기 함수인가의 차이이다.

#### mockFn.mockResolvedValueOnce()

```ts
test('async test', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValue('default')
    .mockResolvedValueOnce('first call')
    .mockResolvedValueOnce('second call');

  await asyncMock(); // 'first call'
  await asyncMock(); // 'second call'
  await asyncMock(); // 'default'
  await asyncMock(); // 'default'
});
```

비동기 함수에 한 번의 호출 당 반환 값을 지정해 준다. `mockResolvedValueOnce`를 선언한 횟수보다 많은 횟수를 호출할 경우 `MockResolvedValue`값을 반환한다.

더 많은 Mocking Method가 있지만 자주 사용하는 것 위주로 정리해 보았다. 더 많은 자료를 찾고 싶으면 [JestMockFunction](https://runebook.dev/ko/docs/jest/mock-function-api) 을 참고

### 자주사용하는 Matchers

>Matcher는 테스트의 결괏값을 매칭하거나, 특정 함수가 내가 설정한 인자로 작동했는지, 예외 처리와 함수 실행 횟수 등을 체크해 정확히 일치할 경우에만 Test Case가 통과된다.

#### toBe()

```ts
const human = {
  name: '기석',
  age: 28,
};

describe('human age', () => {
  test('age is 28', () => {
    expect(human.age).toBe(28);
  });

  test('he is name 기석', () => {
    expect(human.name).toBe('기석');
  });
});
```

객체의 인스턴스 값 또는 원시값을 비교한다. 소수점의 경우 비교가 불가능하다(Js 에서 소수점 계산은 결괏값이 다르다)

#### toEqual()

```ts
const human1 = {
  name: '기석',
  age: 28,
};
const human2 = {
  name: '기석',
  age: 28,
};

describe('두명의 사람이 있다.', () => {
  test('두 사람은 이름과 나이가 같은가', () => {
    expect(can1).toEqual(can2);
  });
});
```

객체의 인스턴스의 모든 속성을 재귀적으로 비교한다. 원시값을 비교하는 `toBe()`와는 다르게 동작한다.

#### rejects.toThrowError()

```ts
it('존재하는 유저를 생성 요청 시 유저가 존재하다는 예외를 던진다.', async () => {
	const result = async () => {
		await userService.createUser(createUserInput);
	};

	expect(result).rejects.toThrowError(
		new HttpException('User already exists', HttpStatus.CONFLICT),
	);
});
```

Test 실행 중 예외를 던졌는지 확인한다. 예외를 던지면 더 이상 코드 실행이 되지 않기 때문에 `expect()` 함수 안에서 실행시켜야 한다.

#### toHaveBeenCalledTimes()

```ts
test('각자 다른 음료수의 갯수', () => {
  const drink = jest.fn();
  drinkEach(drink, ['lemon', 'octopus']);
  expect(drink).toHaveBeenCalledTimes(2);
});
```

해당 함수가 호출된 횟수를 확인해 비교한다. 

#### toHaveBeenCalledWith()

```ts
test('registration applies correctly to orange La Croix', () => {
  const beverage = new LaCroix('orange');
  register(beverage);
  const f = jest.fn();
  applyToAll(f);
  expect(f).toHaveBeenCalledWith(beverage);
});
```

해당 함수가 호출될 때 사용된 인자를 확인해 비교한다.

## 마치며

오늘은 Test 기법의 하나인 Unit Test를 알아보았다. TDD(테스트 주도 개발)이란 것이 존재 하기도 하며. CI/CD의 프로세스 중 하나가 Test Case를 실행 시킨 후 배포가 진행되기 때문에 올바른 테스트코드는 중요하다.
오늘 Unit Test를 배우며 이 점을 더욱 느끼게 되었고, 다음엔 E2E(기능 테스트)를 배워 보려고 하며, 기회가 된다면 TDD또한 접해보고 싶다.