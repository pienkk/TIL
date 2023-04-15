>Prisma는  차세대 Node.js와 Typescript ORM이다. 
>작관적인 데이터모델, 자동화된 마이그레이션, 타입안정성과 자동완성 기능으로 새로운 차원의 개발경험을 선사한다. (공식 홈페이지 번역)
>저번 포스팅에서 Prisma의 인기가 높아지는 이유에 대해 알아봤다. 오늘은 Prisma를 이용해 DB연결과 테이블 생성, 간단한 사용 방법을 알아보고자 한다.

## 설치 / DB 연결

Prisma를 사용하기 위해선 `@prisma/client` npm 패키지를 설치해야한다. 패키지를 설치하면 기본적으로 `node_modules/.prisma/client`  경로에 prisma가 설치된다.

```bash
$ npm install @prisma/client
```

설치가 완료되면 프로젝트가 있는 경로의 터미널에서 `prisma init`를 입력하면 `prisma` 폴더가 생성된다.

```bash
$ prisma init
```

기존에 이미 생성된 DB Schema가 있다면, `prisma db pull`를 입력하면 기존 Schema대로 `prisma/schema.prisma`파일이 동기화된다.
기존에 생성된 DB가 없다면 `prisma/schema.prisma` 파일을 기존 ORM과 같이 Schema를 선언해줘야한다.

### DB Schema 작성
```js
generator client {
	provider = "prisma-client-js"
	binaryTargets = ["native", "rhel-openssl-1.0.x"]
}

datasource db {
	provider = "mysql"
	url = env("DATABASE_URL")
}

model User {
	id Int @id @default(autoincrement()) @map("user_id")
	email String @unique
	password String
	name String
	createdAt DateTime @default(now()) @map("created_at")
	posts Post[]
}

model Post {
	id Int @id @default(autoincrement()) @map("post_id")
	title String
	content String
	published Boolean
	authorId Int @map("author_id")
	author User @relation(fields: [authorId], references: [id], onDelete: Cascade)
	createdAt DateTime @default(now()) @map("created_at")
	comments Comment[]
}

model Comment {
	id Int @id @default(autoincrement()) @map("comment_id")
	content String
	postId Int @map("post_id")
	post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
	createdAt DateTime @default(now()) @map("created_at")
}
```

`DATABASE_URL`은 `.env` 파일에 `DATABASE_URL="mysql://USERNAME:PASSWORD@HOST:PORT/DBNAME"`를 입력해주면 환경변수로 들어가며, DB의 정보를 기입해주면 된다.
테이블마다 model로 구분해 생성하고, 서비스단에서의 키값과 DB상의 키값을 별개로 하고 싶을땐 서비스단에서의 키값을 작성한 뒤 `@map()`안에 DB에서 사용될 키값을 작성하면 키값이 매칭되어 작성된다.
그 외 `@default()`, `@unique`와 같이 DB의 기본값과 제약을 거는등의 행위도 가능하며,
관계 작성의 1:N 관계의 경우 1에 해당하는 model에 N이 들어갈 키값과 모델을 값으로 작성해 준다. 그 후 N에 해당하는 model에서 `@relation(fields:[], reference:[])`를 입력해 관계를 설정해 주면 prisma의 Schema작성이 끝난다.

### Prisma 연결

Schema를 불러오거나 생성했다면 터미널에서 `prisma generate`를 입력해준다. Prisma는 Schema외에 코드상으로 테이블 구조가 정의 되어 있어 해당 명령어는 Schema가 변경될 때마다 입력해줘야한다. 

```bash
$ prisma generate
```

## Prisma 사용

```ts
// prisma/context.ts
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default prisma;
```

prisma를 사용하기 위해선 `PrismaClient()` 를 이용해 객체를 만들고 해당 객체를 통해 prisma를 사용할 수 있다.

```ts
import { CreateUserDto } from "../dtos/createUserDto";
import prisma from "../../prisma/context";
import { GraphQLError } from "graphql";
import { User } from "@prisma/client";

export class UserService {
	// 유저 생성
	public async createUser(createUserDto: CreateUserDto): Promise<User> {
		// 유저 가입 여부 확인
		const user = await prisma.user.findFirst({
			where: { email: createUserDto.email },
		});

		// 이미 가입한 유저인 경우
		if (user) {
			throw new GraphQLError("이미 가입된 유저입니다.", {
				extensions: {
					code: "BAD_USER_INPUT",
				},
			});
		}

		// 유저 생성 후 생성된 유저 반환
		return await prisma.user.create({
			data: {
				email: createUserDto.email,
				password: createUserDto.password,
				name: createUserDto.name,
			},
		});
	}
}
```

prisma에서 DB테이블 객체에 접근하기 위해선 위에서 생성한 prisma객체에 이전에 생성한 Schema에서 만든 모델명을 이용해 prisma ORM을 사용할 수 있다.
위 코드는 회원가입 예시이다. user테이블에 `findFirst`메서드로 email의 중복여부를 검사하고, 중복되지 않았다면 `create`메서드를 통해 유저를 생성하고 생성된 유저 객체를 반환하게 된다.
`findFirst`는 조건에 일치하는 첫번째 값을 반환하고, `create`는 입력된 값으로 DB에 저장하고, 해당 값을 반환해준다.

```ts
import { CreatePostDto } from "../dtos/createPostDto";
import prisma from "../../prisma/context";
import { GraphQLError } from "graphql";
import { Comment, Post } from "@prisma/client";

export class PostService {

	public async getPostsByUser( name: string ): Promise<(Post & { comments: Comment[] })[]> {

		// 유저 확인
		const user = await prisma.user.findFirst({ where: { name } });

		// 일치하는 유저가 없는 경우
		if (!user) {
			throw new GraphQLError("정보가 일치하는 유저가 없습니다.", {
				extensions: {
				code: "BAD_USER_INPUT",
			},
		});
	}

		// 출판되고, 제목에 graphql이 포함된 게시글 리스트 반환
		return await prisma.post.findMany({
			where: {
				AND: [{ authorId: user.id }, { published: true }],
				title: { contains: "graphql" },
			},
			include: {
				comments: true,
			},
		});
	}
}
```

위 코드는 특정 유저의 게시글을 조회하는 API다. 유저 유무를 체크한 뒤 해당 유저가 작성한 게시글 리스트를 반환하는데, 조건으로 published가 true인 값과 제목에 graphql이 포함된 게시글을 검색한다.
기존 TypeORM같은 경우 LIKE를 통해 해당 기능을 사용했지만 prisma는 `contains`를 통해 LIKE와 같은 결과를 낸다.
그리고 연관관계의 댓글을 같이 불러와 반환하게 된다.

## 마치며
>TypeORM에 먼저 익숙해져있는 상태에서 Prisma를 접하게 됐는데 상당부분이 다르게 느껴졌다.
>TypeORM은 SQL에 가까운 ORM이라 생각되는데, Prisma는 좀더 프로그래밍 언어에 가까운 ORM이라고 느껴졌다.
>ORM을 처음 접하게 된다면 Prisma는 매우 좋은 선택지가 될것이라 생각된다.
>또한 TypeORM에선 보통 DI를 통해 DB로직을 수행하는데, Prisma는 싱글톤 방식으로 DB로직을 수행해 새롭게 느껴져서 좋았다.
>나 또한 빠르게 발전해나가는 Prisma를 익혀 큰 프로젝트에도 사용해볼 날이 머지 않을것이다.

