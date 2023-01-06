>Nest.js는 백엔드 서버를 구축하기 위한 Node.js 프레임 워크이다.
TypeScript를 지원하며, 순수 JavaScript로도 사용가능하다.

기존 3개의 프로젝트를 Express를 이용해 진행한 뒤, 이번에 새로운 프로젝트를 Nest.js를 이용해 진행하게 되었다. 앞으로 프로젝트를 진행하며 느낀점과 Nest.js의 특징들을 정리해 보고자 한다.

## Express 프레임워크의 장점 및 단점
Node.js를 이용한 백엔드 서버는 Express가 보편적으로 사용되며, 가장 높은 점유율을 보유하고 있다. 또한 기본적인 서버를 구축하기 매우 쉽게 이루어져있고, 확장성이 좋으며, 자유롭다.
하지만, 이러한 장점들은 단점으로 다가 올 수가 있는데, 너무 자유롭기 때문에 서버를 만드는 사람들 마다 작성법이 다르다는 것이다. 이는 코드의 생산성 저하, 유지보수에 단점으로 다가온다.

## Nest.js를 사용하는 이유
>Java 언어의 Spring 프레임워크를 사용해본적이 있다. 프로젝트를 생성하는 순간 기본적인 틀이 완성 되는 것을 보고, 감탄을 했다. 

Spring 처럼 기본적인 틀이 있고, 규칙과 제약이 있는 Node.js의 프레임워크 그게 바로 Nest.js다.
Nest.js는 Node.js를 사용하며 기본적인 규칙과 제약을 공유함으로 다양한 프로젝트를 진행하더라도, 생산성이 떨어지지 않으며, 유지보수의 증가로 이어질 것이다.
그리고 마크도 멋지다..

### 안정성

Nest.js는 Typescript를 완벽히 지원한다. 이는 개발단계에서 발생하는 유저에러를 방지할 수 있다.

## Nest.js의 구조

### Modules

>모듈은 `@Module()` 이 데코레이터로 달려있는 클래스를 의미한다. API당 하나 이상의 모듈이 존재하며, 최상단에 appModule이 있다. 각 모듈은 공급자 관계와 종속성을 가지고 있으며, 모듈은 해당 API에서 사용할 기능들을 가져오거나 내보낸다.

#### Nest.js 모듈 관계도

![](https://velog.velcdn.com/images/kisuk623/post/ba512df0-bc01-4a1d-a8ba-1adc475a3ad9/image.png)

위 사진과 같이 Nest.js의 각 모듈들은 하위 모듈에 대해 종속성을 가지고 있다. 

#### 기본적인 Module의 형태

```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
	imports: [DogModule],
	controllers: [CatsController],
	providers: [CatsService],
	exports: [CatsService]
})

export class CatsModule {}
```

| imports     | 다른 모듈에서 export한 모듈을 현재 모듈에서 사용할 목록 |
| ----------- | ------------------------------------------------------- |
| controllers | 이 모듈에서 사용할 컨트롤러                             |
| providers   | 이 모듈에서 사용할 공급자                               |
| exports     | 다른 모듈에서 사용할 공급자                                                      |

### Controller

>컨트롤러는 @Controller() 데코레이터가 달려있는 클래스를 의미한다. 컨트롤러는 들어오는 request를 처리하고, response를 클라이언트에게 반환한다.

#### Controller 관계도

![](https://velog.velcdn.com/images/kisuk623/post/e44dab11-696f-4ed5-913d-758118f2eb65/image.png)


#### 기본적인 Controller의 형태

```ts
import { Controller, Get, POST } from '@nestjs/common';

@Controller('cats')
export class CatsController {
	@Get()
	findAll() {
		return 'This action returns all cats';
	}

	@POST()
	createCat(){
		return 'This action create cat'
	}
}
```

`@Controller('cats')`와 `@GET()`데코레이터는 Express 프레임워크의 Route기능에 해당한다. `/cats` 엔드포인트로 GET 요청을 보낼경우 `findAll` 함수가 실행되며, POST 요청을 보낼경우 `createCat` 함수가 실행된다.

### Provider

>공급자는 @Injectable() 데코레이터가 달려있는 클래스를 의미한다. 공급자는 Nest의 기본 개념이다. 공급자의 주요 개념은 의존성을 주입 시킨다는 것에 있으며, 개체마다 서로 다양한 관계를 만들어 Nest런타임 시스템에 해당 기능을 위임시킨다.

#### Provider 관계도

![](https://velog.velcdn.com/images/kisuk623/post/b2c9fdc6-24c2-4b69-8e3c-c92353273c9f/image.png)


#### 기본적인 Service의 형태
```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
	private readonly cats: Cat[] = [];

	create(cat: Cat) {
		this.cats.push(cat);
	}

	findAll(): Cat[] {
		return this.cats;
	}
}
```

Service의 경우 Nest나 Express만이 사용하는게 아닌 대부분의 프레임워크에서 공통적으로 사용한다. 해당 레이어는 서버의 각종 로직을 담당한다.

### Dependency Injection
>Nest.js는 종속 주입으로 알려진 패턴을 중심으로 구축되었다. Typescript의 기능 덕분에 종속성이 유형별로 해결되어 Nest.js는 종속성을 관리하기가 매우 쉽다.

#### 종속성을 사용한 Controller
```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
	constructor(private readonly catsService: CatsService) {}

	@Post()
	async create(@Body() createCatDto: CreateCatDto) {
		this.catsService.create(createCatDto);
	}

	@Get()
	async findAll(): Promise<Cat[]> {
		return this.catsService.findAll();
	}
}
```
생성자 함수로 `CatsService` 인스턴스를 Typescript의 `private` 접근 제어자를 통해 생성하고 반환한다. 이를 통해 `CatsController` 클래스는 `CatsService`  클래스를 종속 주입하게 되며, `CatsService`의 인스턴스를 사용할 수 있게 된다.

## Nest.js 설치 & 실행

Nest.js로 프로젝트를 만들기 전에 nestjs/cli를 설치 해야한다. 터미널에 다음 명령어를 입력하라
```bash
$ npm i -g @nestjs/cli
```

@nestjs/cli 를 설치 한 후, 터미널에 `nest` 를 입력해 보자. 상당히 많은 명령어가 나오는데, 프로젝트를 생성하는 명령어는 `nest new projectName` 이다.
```bash
$ nest new projectName
```

해당 명령어를 입력 하면 패키지 매니저로 어떤것을 사용할 것인가 물어본다. 나는 npm을 사용할 것이다. 패키지 매니저를 선택하고 조금 기다리면 프로젝트 이름의 폴더가 생성되고, `eslint`, `prettier`와 같은 기초 세팅이 완료되어있다.

즉시 서버도 실행 시킬 수 있으며 기본적으로 포트는 3000에 할당되어 있다.
기존 express를 사용할 때에는 `nodemon` 을 설치해 사용했다. `nodemon`은 코드의 변화가 생기면 서버를 리부팅해 개발 환경을 좀더 쾌적하게 해준다. Nest.js는 이를 기본적으로 제공한다.

```bash
## 기본 실행
$ npm run start

## 변경 감지 서버 리부팅
$ npm run start:dev
```

## Nest.js 프로젝트 세팅

터미널에 nest를 입력하면 수많은 명령어가 나오는데 해당 명령어를 사용하면 손쉽게 프로젝트를 구성 할 수 있다.

```bash
## 유저 모듈 생성
$ nest g mo users

## 유저 컨트롤러 생성
$ nest g co users

## 유저 서비스 생성
$ nest g s users
```

해당 명령어를 터미널에 입력 하면 /src 폴더에 /users 폴더가 추가된 것을 알 수 있다. 폴더에는 모듈, 컨트롤러, 서비스가 추가되어있으며, 모듈이 자체적으로 연결되어 있다.
이처럼 Nest.js는 프로젝트 세팅이 매우 간편하다.

>Nest.js의 기본적인 틀과 기초적인 설치방법을 알아 보았다. Nest.js는 기본적으로 express혹은 fastapi 프레임워크 위에서 돌아간다. express 위에서 돌아간다 함은 오리지널 express 보다 속도가 느리지 않을까 걱정이 될 수도 있지만, 결국 패키지를 많이 설치할 경우 속도는 거의 동일하다고 한다. 
다음엔 구조를 더 자세히 알아보도록 하자.
또한 Nest.js의 공식문서는 매우 친절한 편이니 찾아보도록 하자.
