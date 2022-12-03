## Typescript란?
> Typescript란 Javascript의 상위 확장 언어로 MS에서 만들어 졌다.
Typescript의 코드를 실행 시키면 Typescript의 코드는 Babel을 이용해 Javascript상의 원하는 버전의 문법으로 컴파일 된다.

## Typescript를 쓰는 이유
>Javascipt는 동적 언어다. 동적 언어는 코드를 실행할 때(런타임) 타입을 결정하는 언어를 뜻한다.
동적 언어의 장점은 유연성 있는 개발이 가능하다는 점이 있지만, 내가 원하지 않는 타입과 예상치 않은 오류로 인해 코드의 안정성이 떨어진다는 단점이 있다.
Typescript는 정적 언어다. 코드를 작성하는 순간 타입을 지정해 주고, 타입 에러가 날 경우 컴파일 단계에서 잡아준다.

```js
let arr = [1,2,3,4,5]
arr += 2
console.log(arr) // '1,2,3,4,52'
```
Javascript는 위 처럼 배열에 다른 타입의 값을 더해줬더니 문자열 타입으로 변했다. 실수로 해당 연산을 실행시켰더라고 하더라도 우리는 배열을 잃는것을 원치 않을 것이다. 
Typescript에서 해당 코드를 작동 시켰을 경우, 타입 에러가 나타난다. 배열에 숫자 타입 언어는 연산이 되지 않는다.

---
## Type

### Object

```tsx
const player : {
name : string,
age? : number
} = {
name: "gisuk"
}
```

Typescript는 값 마다 타입을 지정 해 줄 수 있으며,타입을 지정해 주지 않을 경우, Typescript가 타입을 직접 추론해 할당 해 준다. 특정 값의 유무가 확실하지 않을 경우 해당 값에 **?** 를 붙이면 선택 유무 타입으로 사용할 수 있다.


### Type alias

```tsx
type Player = {
name: string,
age?: number
}

const gisuk : Player = {
name: "gisuk"
}

const juwon : Player = {
name: "juwon",
age : 33
}

type Words = {
[key:string]: string
}

const dict: Words = {
"potato": "food",
"tomato": "food
}
```

같은 타입의 객체를 만들게 될 경우 객체의 타입을 선언 해서 사용 할 수 있다. Type alias(별칭) 이라고 부른다.
객체의 `key` 값을 고정시키지 않고, 타입지정을 할 경우, `[]`  로 감싸면 된다.


### readonly

```tsx
const numbers: readonly number[] = [1,2,3,4];

numbers.push(1) // 에러 발생
```

배열의 값을 고정시키고 싶을 때 readonly를 사용하면 배열의 값이 변경 불가능하다.


### Tuple

```tsx
const player : [string, number, boolean] = ["2",1,true]

player[0] = 1; // 에러 발생
player[1] = 3; // 정상 작동
```

배열의 특정 인덱스의 값을 지정할 수 있는데, 이를 Tuple 이라고 한다. Tuple로 지정해둔 타입 이외의 값으로 수정하려고 할 경우 에러가 발생한다.


### any, unknown

```tsx
let a: unknown;

if(typeof a === 'number'){
	let b = a + 1
}

if(typeof a === 'string'){
	let b = a.toUpperCase();
}
```

`any` 와 `unknown` 은 비슷하지만 다른 특성을 가지고 있다.
두 타입 모두 모든 타입을 허용한다는 점에서 동일 하지만, `any`는 타입스크립트를 완전히 벗어나 예상치 못한 오류를 발생 시킬수 있다. 
하지만 `unknown` 은 타입스크립트를 벗어나는게 아니다. `unknown`을 특수한 메소드와 같이 사용할 경우, 메소드에 대한 타입을 조건으로 걸어 준 뒤에 사용하면 타입스크립트는 정상적으로 작동 한다.


### 특정 문자만 받기

```tsx
type Team = "red" | "blue" | "yellow"
type Health = 1 | 5 | 10

type Player = {
nickname: string,
team:Team,
health: Health
}

const gisuk: Player = {
nickname: "gisuk",
team: "red",
health: 15 //에러 발생
}
```

특정 변수 혹은 객체의`key` 값에 특정한 값만 받고 싶을 때가 있다.
`| (OR)`  을 사용해 특정 값을 넣어주면 해당 타입은 그 특정 값만 저장 할 수 있다.

---

## Object

### call signature

```tsx
function add(a:number, b:number) {
	return a + b
}

---

type Add = (a:number, b:number) => number;

const add2:Add = (a,b) => a+b;
```

타입스크립트의 함수는 매개변수의 타입을 지정해 줘야 작동한다.
함수의 매개변수 타입이 같은 함수를 생성할 경우, 불필요한 타입 지정을 계속 하게 되는데, `call signature` 를 사용해 함수의 타입을 선언해서 사용 하면 함수를 생성 할 때 마다 타입을 지정해야 하는 반복 작업을 줄일 수 있다.


### overloading

```tsx
type Config = {
path: string,
state : number
}

type Push = {
(path: string): void
(config: Config): void
}

const push: Push = (config) => {
if(typeof config === "string") {console.log(config)}
else{console.log(config.path, config.state)}
}
```

한가지 함수에 여러가지 매개 변수를 사용 하고 싶을 때가 있다.
매개 변수의 타입에 따라 함수의 실행을 다르게 하고 싶은 경우 `overloading` 을 사용한다. 함수에 대한 타입을 정해 둔 뒤, 함수에 해당 타입과 조건문을 통해 실행 결과를 다르게 해 주면 매개 변수의 타입에 따라 다른 결과를 도출 해 낼 수 있다.
`overloading` 은 `type` 대신 `interface` 를 사용 할 수도 있다.

```tsx
type Add = {
	(a:number, b:number) : number;
	(a:number, b:number, c:number) : number;
}

const add:Add = (a, b, c?:number) => {
	if(c) return a + b + c
	return a + b
}
```

매개변수의 개수에 따라서도 다른 실행 결과를 원할 경우도 있다. 이럴때 에도 `overloading` 을 사용한다.

---

## Generic


>매개 변수의 타입에 따라 함수의 실행 값을 변경 할경우 overloading 을 사용했다. `overloading` 과 같은 방식을 콘크리트 타입 이라고 한다. 하지만 경우의 수가 계속 늘어 나거나 값의 타입이 변할 경우 코드의 재 사용성과 유지보수에 좋지 않다. 이럴 때 사용하는게 `generic` 이다.

### generic
```tsx
type SuperPrint = {
	<T>(arr: T[]): T
}

const superPrint: SuperPrint = (arr) => arr[0]

superPrint([1,2,3,4])

superPrint([true,false])

superPrint([1,2,"a",true])

---

function superPrint2<V>(a: V[]){
	return a[0]
}
```

`generic` 을 사용하면 타입스크립트가 타입을 유추해 주며 이는 `call signature` 를 대체해 준다.
해당 변수에 타입을 추론 해 주고 입력값, 출력값의 타입 모두 `generic` 이 타입 추론을 해준다.


```tsx
type SuperPrint = <T,M>(a: T[], b:M) => T

const superPrint: SuperPrint = (a,b) => a[0]

superPrint([1,2,3,4],"x")

superPrint([true,false],1)

superPrint([1,2,"a",true],false).toUpperCase() // 에러 발생

superPrint(["a"],true).toUpperCase()
```

`generic` 은 복수의 변수에도 사용 가능하며, 매개변수의 순서에 따라 타입지정이 이뤄진다.

`generic` 과 `any` 는 언뜻 보면 비슷하다. 하지만 `generic` 은 타입을 추론 해 줄 뿐 `any` 처럼 타입스크립트를 벗어가는게 아니다.
`any` 타입이 지정된 배열에 `toUpperCase()` 메소드를 사용할 경우, 배열에 어떠한 값이 들어가 있어도 에러가 발생하지 않지만 `generic` 을 사용했을 경우, 에러가 날 가능성이 있을 경우 타입 에러를 발생 시킨다. 

```tsx
type Player<E> = {
	name:string
	extraInfo:E
}

const gisuk: Player<{favFood:string}> = {
	name:"gisuk",
	extraInfo: {
		favFood: "kimchi"
	}
}
```

콘크리트 타입과 제네릭을 같이 사용할 경우 위 처럼 작성하면 된다.

```tsx
type Player<E> = {
	name:string
	extraInfo:E
}

type GisukPlayer = Player<{favFood: string}>

const gisuk: GisukPlayer = {
	name:"gisuk",
	extraInfo: {
		favFood: "bullgogi"
	}
}

const juwon: Player<null> = {
	name: "juwon",
	extraInfo:null
}
```

확장성 타입을 지정해 위 처럼 사용도 가능하며, 타입을 나누는 이유는 재 사용성을 위해 타입을 나눠 더 효율성 있는 코드로 만드는 것이다.



## Class


### class, 접근 제어자

```tsx
class Player {
	constructor(
	private firstName:string,
	private lastName:string,
	public nickname:string
	){}
}
/* javascript
class Player {
	constructor(firstName, lastName, nickname) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.nickname = nickname;
	}
}
*/

const gisuk = new Player("gisuk","jang","기석");
gisuk.firstName // 에러 발생
gisuk.nickname
```

타입스크립트는 `constructor` 매개변수로 접근제어자와 타입을 지정해주면 타입스크립트가 알아서 `constructor` 의 `this` 선언까지 해 준다.

접근제어자는 `public`, `protected`, `private` 세 가지가 있다.

| 구분      | 선언한 클래스 내 | 상속받은 클래스 | 인스턴스 |
| --------- | ---------------- | --------------- | -------- |
| private   | O                | X               | X        |
| protected | O                | O               | X        |
| public    | O                | O               | O        |


### 추상화 class

```tsx
abstract class User {
	constructor(
		private firstName:string,
		private lastName:string,
		public nickname:string
	){}
	getFullName(){
	return `${this.firstName} ${this.lastName}`
	}
}

class Player extends User { }

const gisuk = new Player("gisuk","jang","기석");
const jang = new User("gisuk","jang","기석") // 에러발생

gisuk.nickname
gisuk.getFullName()
```

추상화 `class` 는 일반 `class` 처럼 사용이 불가능하다.
추상화 클래스는 다른 클래스에 상속하는 용도로만 사용 가능하며, 추상화 클래스로 생성자 함수를 사용할 경우 타입스크립트 상에서 에러가 발생한다.


### interface

```tsx
interface User {
	name:string
}
interface Player extends User {
}
const gisuk: Player = {
	name :"기석"
}
///---------------------//
type User = {
	name:string
}
type Player = User & {
}
```

`interface` 는 객체의 타입을 지정 해 준다는 점에서 Type문법과 매우 유사하다. 기능은 Type이 더 많고 문법이 살짝 다르다.
Type 이 꼭 필요한 경우가 아니면 `interface`를 사용하자.


### class 변수 초기화

```tsx
type Words = {
	[key:string]: string
}
class Dict {
	private words: Words
	constructor(){
		this.words = {}
}
	add(word: Word) {
	if(this.words[word.term] === undefined){
		this.words[word.term] = word.def;
	}
}
	def(term:string){
		return this.words[term]
	}
}

class Word {
	constructor(
		public term:string,
		public readonly def:string
	){}
}

const kimchi = new Word("kimchi", "한국의 음식");
const dict = new Dict();
  
dict.add(kimchi)
dict.def("kimchi")
```


`constructor` 접근 제어자를 활용해 매개 변수를 지정할 경우 타입스크립트에서 자동으로 `this` 바인딩을 해준다. 하지만 이렇게 작성할 경우, 생성자 함수를 통해 `class` 를 생성 할 때 인자를 추가해 줘야한다.
매개변수를 추가하지 않고, `class` 내부에 변수를 만들려면 먼저 선언을 하고 `this` 를 직접 기입해 주면 된다.


### 추상화 class 와 interface

```tsx
abstract class User {
	constructor(
		protected firstName:string,
		protected lastName: string
	){}

	abstract fullName(): string;
	abstract sayHi(name:string): string;
}

class Player extends User {
	fullName() {
		return `${this.firstName} ${this.lastName}`;
	}

	sayHi(name:string) {
		return `Hello ${name}. My name is ${this.fullName()}`
	}
}
```

```tsx
interface User {
	firstName:string,
	lastName:string
	sayHi(name:string):string
	fullName():string
}

class Player implements User {
	constructor(
		public firstName:string,
		public lastName:string
	){}

	fullName(){
		return `${this.firstName} ${this.lastName}`
	}
	sayHi(name:string){
		return `Hello ${name}. My name is ${this.fullName()}`
	}
}
```

위 두 가지 코드는 동일하게 작동한다.
추상화 `class` 는 자바스크립트로 컴파일 할 때 일반 `class` 로 남게 되고, `interface` 는 상속받은 클래스만 남고 `interface`는 사라진다.

추상화 `class`는  `constructor` 에 접근 제어자를 사용해 `this` 바인딩을 할 수 있지만, `interface` 는 불가능 하다. 
추상화 `class` 는 상속받은 `class` 에 `constructor` 를 사용하지 않아도 되지만, `interface` 는 사용을 해야 작동을 하며, `public`  접근 제어자만 사용 할 수 있다.


### generic

```tsx
interface SStorage<T> {
	[key:string]: T
}

class LocalStorage<T> {
	private storage: SStorage<T> = {}

	set(key:string, value:T){
		this.storage[key] = value
	}
	get(key:string): T{
		return this.storage[key]
	}
	remove(key:string) {
		delete this.storage[key]
	}
	clear(){
		this.storage
	}
}

const stringStorage = new LocalStorage<string>();

stringStorage.set("gisuk", "good")
stringStorage.get("gisuk")

const booleanStorage = new LocalStorage<boolean>();

booleanStorage.set("true", false)
```

`interface` 와 `class` 에 `generic` 을 사용하면, 변동성 있는 `class` 를 생성 할 수 있다.
`generic` 으로 선언한 `class` 를 생성자 함수로 생성할 때 `generic` 값으로 `Type` 을 지정 해 주면, 해당 `Type` 을 매개 변수로 갖는 `class` 가 만들어진다.

## 마치며
>오늘은 Typescript의 기초 지식에 대해 알아보았다.
Javascript를 사용하며, 내가 원하지 않았던 타입으로 바뀌어 곤란했던 적이 있는데, Typescript를 사용해 보며 매우 매력적인 언어라는 생각이 들었다.
소규모 프로젝트에서는 Typescript의 장점이 크게 드러나지는 않지만, 프로젝트가 커짐에 따라 코드의 안정성을 높여주는 Typescript는 거의 필수 요소 처럼 여겨지고 있다.
앞으로의 프로젝트는 Typescript를 사용해 진행하며 좀더 Typescript 공부를 해 볼 예정이다.