>현재 Node.js에서 주로 사용되는 ORM은 Sequelize, TypeORM, Prisma가 있으며, 주간 npm다운로드 수는 위에 나열한 순서 대로 다운로드 수가 많다.
>오늘은 Typescript에 최적화된 Prisma와 TypeORM의 차이점을 알아보고자 한다.

![](https://velog.velcdn.com/images/kisuk623/post/e2d24fd0-8a87-4d3b-9e94-e1f15225523d/image.png)


## 인기만점 Prisma

>Prisma는 1년전만해도 npm다운로드 수가  TypeORM의 절반에 미치지 못했지만, Prisma의 지속적인 인기의 증가로 2023년 현재 TypeORM과 Prisma의 다운로드 수는 10%정도 차이가 날 정도로 격차가 줄어 들었다.

Prisma와 TypeORM은 같은 ORM이지만 추상화 수준이 다르게 작동 한다. TypeORM은 보다 SQL에 가깝게 설계되어 있고, Prisma는 개발자의 작업을 염두해 더 높은 추상화 수준을 제공 한다.
그렇다고 Prisma로 raw query를 날리지 못한다는 것은 아니다. 필요에 따라 raw query또한 날릴수 있다.

## anyORM??

>TypeORM의 대표적인 별명중 하나는 anyORM이다.
>이름이 TypeORM인데 왜 별명은 anyORM일까? 이는 타입체크를 제대로 해주지 않아, 컴파일 환경에서 에러를 검출하지 못하고, 런타임 환경에서 에러가 발생하게 되는 경우가 생기기 때문 이다.

TypeORM은 Entity를 만들어 DB의 각 테이블을 객체로 추상화해 사용한다. Entity라는 클래스가 있는데 왜 타입체크를 해주지 못한다는 것일까? Prisma와 TypeORM을 비교해보면서 알아보자

## Prisma와 TypeORM의 차이점

### DB 테이블

![](https://velog.velcdn.com/images/kisuk623/post/abf498f8-fea5-4299-8ca7-ad9cf041bc62/image.png)


### TypeORM Entity

```ts
// entity.ts

@Entity()
export class User {
	@PrimaryGeneratedColumn()
	id: number

	@Column({ nullable: true })
	name: string

	@Column({ unique: true })
	email: string

	@OneToMany((type) => Post, (post) => post.author)
	posts: Post[]
}

@Entity()
export class Post {
	@PrimaryGeneratedColumn()
	id: number

	@Column()
	title: string

	@Column({ nullable: true })
	content: string

	@Column({ default: false })
	published: boolean

	@Column()
	authorId: number
	
	@ManyToOne((type) => User, (user) => user.posts)
	author: User
}
```


### Prisma Schema

```ts
// schema.prisma

model User {
	id Int @id @default(autoincrement())
	name String?
	email String @unique
	posts Post[]
}

model Post {
	id Int @id @default(autoincrement())
	title String
	content String?
	published Boolean @default(false)
	authorId Int
	author User @relation(fields: [authorId], references: [id])
}
```

TypeORM과 Prisma는 둘다 유사한 방식으로 Database Schema를 작성한다. 차이점이 있다면 TypeORM은 `ts`확장자를 사용하고 `Prisma`는 prisma 확장자를 사용해 Schema를 작성한다.

### 타입체크

#### TypeORM

```ts
const postRepository = getManager().getRepository(Post)
const publishedPost: Post = await postRepository.findOne({
	where: { published: true },
	select: ['id', 'title'],
})

if (publishedPost.content.length > 0){
	console.log(publishedPost.content)
}
```

위 코드는 컴파일에서 부터 서버가 실행 되기 까지 에러가 발생하지 않지만 `if`조건문을 실행하는 순간 런타임 에러가 발생하게 된다.
`select` 조건에서 `id`와 `title`만 가져와 publishedPost객체에는 content 키값이 존재하지 않는다. 이 때문에 content를 읽지 못해 에러가 발생한다.
TypeORM은 우리가 특정 Column값을 조회하고자 해도 컴파일 단계에서는 완성된 객체로 인식해 위와 같은 오류를 잡아내지 못한다. 이는 런타임 환경에서 에러가 발생할 수 있다는 아주 큰 문제를 가지고 있다.

```node
TypeError: Cannot read property 'length' of undefined
```

#### Prisma

```ts
const publishedPost = await prisma.post.findFirst({
	where: { published: true },
	select: {
		id: true,
		title: true,
	},
})

if (publishedPost.content.length > 0) {
	console.log(publishedPost.content)
}
```

TypeORM과 달리 Prisma는 타입의 안전을 보장한다. 특정 Column만 조회할 경우 해당 변수는 조회한 Column들로만 구성된 객체가 된다.
`publishedPost`변수는 `id`와 `title` 두개의 키값을 가지는 객체인 것이다.
위코드를 컴파일 할 경우 `content`를 찾을수 없다는 에러가 발생해 컴파일이 진행되지 않아 런타임 에러의 가능성을 배제해준다.

```node
[ERROR] 14:03:39 ⨯ Unable to compile TypeScript:

src/index.ts:36:12 - error TS2339: Property 'content' does not exist on type '{ id: number; title: string; }'.
```


### 연관 테이블(Relation table)

#### TypeORM

```ts
const postRepository = getManager().getRepository(Post)
const publishedPosts: Post[] = await postRepository.find({
	where: { published: true },
	relations: ['author'],
})
```

TypeORM은 연관 테이블을 정의 할때, 특정 Column을 가져올 때 문자로 정의를 한다. 문자열로 정의 하기 때문에 자동완성 기능의 도움을 받지 못하고, 연관 테이블을 잘못 입력해도 컴파일 과정에서 에러가 발생하지 않는다.

```ts
const publishedPosts: Post[] = await postRepository.find({
	where: { published: true },
	// author을 authors로 잘못입력
	relations: ['authors'],
})
```

위 코드는 컴파일 에러가 나지 않고 서버 실행시 까지 에러가 나지 않는다.
해당 함수를 실행하는 순간 연관 테이블을 찾지 못한다는 런타임 에러가 발생하게 된다.

```node
UnhandledPromiseRejectionWarning: Error: Relation "authors" was not found; please check if it is correct and really exists in your entity.
```


#### Prisma

```ts
const publishedPosts = await prisma.post.findMany({
	where: { published: true },
	include: { author: true },
})
```

Prisma는 연관 테이블을 정의하거나, 특정 Column을 가져올 때 키값으로 정의 한다. 덕분에 자동완성 기능의 도움을 받을수 있고, 오타가 날 경우 바로 에러를 확인할 수 있어, 런타임 에러가 발생하지 않는다.


## 마치며
>TypeORM을 쓰면서 사람들이 왜 anyORM이라고 하는지 처음엔 이해하지 못했다.
>하지만 Prisma와 비교하며 알아본 결과 TypeORM에서 뜨는 오류가 당연히 뜨는 오류라고 생각했던 것이 당연한것이 아니였다... 
>타입체크를 통해 런타임 에러를 최소화 하기 위해 Typescript를 사용한 것인데, 오타하나로 런타임 과정에서 에러가 발생해 버리니 Typescript를 사용한게 말짱 도루묵인 느낌이었다.
>Prisma는 기존에 node에서 존재하는 ORM과는 다른 느낌이 었다. 좀 더 개발자 친화적이고, 휴먼에러의 발생확률을 줄여줘 왜 인기가 점점 많아지는지 알게된 시간이었다.
>오늘은 간략하게 TypeORM과 Prisma의 차이점을 알아 봤는데, 에러에 관련된 내용 위주로 알아 봤다. 다음엔 Prisma에 대해 좀더 자세히 알아보고자 한다.