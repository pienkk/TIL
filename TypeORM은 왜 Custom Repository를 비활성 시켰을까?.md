>TypeORM은 nodeJs 환경에서 사용할 수 있는 ORM중 하나다.
>TypeORM을 제외하고 많이 사용되는 ORM에는 Sequelize, Prisma가 있다.

TypeORM은 현재 0.3.11 버전이다. 22년 3월경 0.2 > 0.3 버전으로 업그레이드가 되었는데 버전이 변경되면서 상당히 많은 것이 변경되었다.

특히 기존 0.2버전을 이용하다가 0.3버전을 이용할 경우 기본적으로 제공하던 Custom Repository 기능을 더 이상 지원하지 않는다. 오늘은 Custom Repository에 대해서 알아보고자 한다.

## Custom Repository

>Repository는 DB에 직접적으로 엑세스할 수 있는 레이어를 말하며, Entity로 정의한 테이블의 CRUD를 관리하는 역할을 한다.
>TypeORM은 DB에 접속하기 위한 `find()`와 같은 메서드들이 기본적으로 존재하지만 복잡한 요청이 필요할 때 Custom Repository를 생성해 직접 메서드를 만들어 사용할 수 있다.

### 0.2버전의 Custom Repository

```ts
// community.service.ts
@Injectable()
export class CommunityService {
	constructor(
		@InjectRepository(Post)
		private postRepository: PostRepository) {}

	getPosts(): Promise<PostListDto> {
		return this.postRepository.getPostLists();
	}
}
```

```ts
// post.repository.ts
@EntityRepository(Post)
export class PostRepository extends Repository<Post> {
	async getPostLists() {
		return await this.find()
	}
}
```

0.2 버전의 방식은 `@EntityRepository` 데코레이터를 Repository에 작성해 준 뒤, Service 레이어에서 Repository를 의존성 주입을 해서 사용했다.
하지만 TypeORM이 0.3버전으로 바뀜에 따라 `@EntityRepository` 데코레이터를 사용할 수 없게 변경되었다.
이에 따라 Custom Repository를 사용하기 위해선 직접 데코레이터를 사용해 Service 레이어로 의존성을 주입해야 한다.

### 0.3버전의 Custom Repository

```ts
// src/config/typeorm/typeorm-ex.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const TYPEORM_EX_CUSTOM_REPOSITORY = 'TYPEORM_EX_CUSTOM_REPOSITORY';

export function CustomRepository(entity: Function): ClassDecorator {
	return SetMetadata(TYPEORM_EX_CUSTOM_REPOSITORY, entity);
}
```

우선 `CustomRepository` 데코레이터를 정의해 준다. `SetMetadata`는 메타데이터를 설정해주는 역할을 한다.

```ts
// src/config/typeorm/typeorm-ex.module.ts
import { DynamicModule, Provider } from '@nestjs/common';
import { getDataSourceToken } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';
import { TYPEORM_EX_CUSTOM_REPOSITORY } from './typeorm-ex.decorator';

export class TypeOrmExModule {
	public static forCustomRepository<T extends new (...args: any[]) => any>(
repositories: T[]): DynamicModule {
		const providers: Provider[] = [];

		for (const repository of repositories) {
		const entity = Reflect.getMetadata(TYPEORM_EX_CUSTOM_REPOSITORY,repository);

		if (!entity) {
			continue;
		}

		providers.push({
			inject: [getDataSourceToken()],
			provide: repository,
			useFactory: (dataSource: DataSource): typeof repository => {
				const baseRepository = dataSource.getRepository<any>(entity);
				return new repository(baseRepository.target,baseRepository.manager,baseRepository.queryRunner);
			},
		});
	}

	return {
		exports: providers,
		module: TypeOrmExModule,
		providers,
		};
	}
}
```

그 후 CustomRepository를 사용하기 위해 모듈을 생성해 준다. `@CustomRepository` 데코레이터를 사용할 경우 CustomRepository에 담겨있는 메타데이터들을 가져온다. 그 후 `DataSource` 정보를 주입받고 `providers` 에 추가한다.
해당 모듈을 통해 우리가 만든 데코레이터로 CustomRepository 기능을 사용할 수 있게 된다.

```ts
// src/community/community.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmExModule } from 'src/config/typeorm/typeorm-ex.module';
import { CommunityController } from './community.controller';
import { CommunityService } from './community.service';
import { PostRepository } from './entity/post.repository';
import { ReplyRepository } from './entity/reply.repository';

@Module({
	imports: [
		TypeOrmExModule.forCustomRepository([PostRepository, ReplyRepository])],
	controllers: [CommunityController],
	providers: [CommunityService],
})

export class CommunityModule {}
```

CustomRepository를 사용하려면 기존 module에서 `import` 부분을 생성해준 모듈로 변경해 줘야 한다.
그리고 해당 서비스에서 사용할 Repository들을 작성해 주면 사용할 수 있다.

```ts
// community.service.ts
@Injectable()
export class CommunityService {
	constructor(private readonly postRepository: PostRepository) {}

	getPosts(): Promise<PostListDto> {
		return this.postRepository.getPostLists();
	}
}
```

```ts
// post.repository.ts
@CustomRepository(Post)
export class PostRepository extends Repository<Post> {
	async getPostLists() {
		return await this.find()
	}
}
```

그 후 Repository에서 `@EntityRepository`  데코레이터를 `@CustomRepository` 로 변경한다.
그리고 Service에서 `@InjectRepository` 데코레이터를 지워준다. 우리는 직접 만든 데코레이터를 사용해서 의존성을 주입하기 때문에 해당 데코레이터가 없어도 정상적으로 작동한다.

### 왜 CustomRepository를 비활성화 시켰을까?

Nest.js와 TypeORM을 사용하는 나를 포함한 대다수의 개발자는 Service 레이어에는 비즈니스 로직만 사용하고, DB에 접근할 때 TypeORM에서 기본적으로 제공하는 메서드를 사용하는 경우에도 CustomRepository를 이용해 레이어를 분리해서 사용했을 것이다.

위 예시 코드가 TypeORM의 기본 메서드를 CustomRepository를 사용한 예시이다. `getPostLists` 함수에는 그저 `find()` 메서드만 실행시키고 그 값을 반환한다. Service 레이어에서 `getPostList()` 와 `find()` 두 가지 메서드를 실행했을 경우 결과는 동일하게 나온다. 이런 경우 굳이 새로운 메서드를 만들 필요가 있는가? 없다고 본다.

이에 관해 여러 개발자들이 의견을 나눈결과 CustomRepository의 무분별한 사용이 좋지 않다고 결과가 나왔고 TypeORM측도 해당 의견에 동의해 CustomRepository기능을 비활성화했다.

## 반성

내가 처음 Nest.js를 접했을 땐 TypeORM의 버전은 이미 0.3버전이었다. Repository 레이어는 필수라고 생각했던 나의 얕은 지식으로는 CustomRepository가 비활성화되었다는 것을 접하고 그냥 무작정 다시 활성화하는 방법을 찾기 시작했고, 데코레이터를 이용해 다시 CustomRepository를 사용했고, 모든 DB에 관련된 코드를 모두 해당 레이어에 작성했다.
하지만 최근에 코드 리뷰를 받으며, CustomRepository가 비활성화된 이유를 듣고 납득함과 동시에 반성하게 되었다.
나는 특정 기능이 비활성화되었다. 그 안에 숨은 뜻을 찾지 못했고, 그 결과 여러 개발자들의 의도와는 다른 코드를 작성하고 있었다.
앞으로는 눈앞에 보이는 코드만 해결하는 것이 아닌 이게 왜 이런 식으로 변경되었고 앞으로 어떤 식의 코드를 작성하면 좋을지 생각하고자 한다.
