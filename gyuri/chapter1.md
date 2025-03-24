## 1.5 이벤트 루프와 비동기 통신의 이해

- 동기(synchronous) 방식 : 직렬 방식으로 작업을 처리하는 것, 한 번에 다양한 많은 작업을 처리할 수 없다.
- 비동기(asynchronous) 방식: 병렬 방식으로 작업을 처리하는 것, 한 번에 여러 작업이 실행될 수 있다.
- 자바스크립트는 싱글 스레드(single thread) 방식으로 동작
  - 싱글 스레드 방식은 한 번에 하나의 작업만 처리할 수 있음

### 1.5.1 싱글 스레드 자바스크립트

- 프로그램이 실행되면 프로세스(process)가 생성되고, 프로세스는 스레드를 생성
- 프로세스(process)란 실행 중인 프로그램을 의미
- 스레드(thread)란 프로그램이 동작할 때 실행되는 흐름의 단위

### 1.5.2 이벤트 루프란?

- 자바스크립트 런타임 V8 기준으로 설명
- ECMAScript, 즉 자바스크립트 표준에 나와 있는 내용은 아니다
- 자바스크립트 런타임 외부에서 자바스크립트의 비동기 실행을 돕기 위해 만들어진 장치

#### 호출 스택과 이벤트 루프

- 호출 스택(call stack)은 함수의 호출을 기록하는 자료 구조
- 호출 스택은 함수가 호출되면 스택에 쌓고, 함수가 종료되면 스택에서 제거

호출 스택이 비어 있는지 여부를 확인하는 것이 바로 이벤트 루프

- 이벤트 루프(event loop)는 호출 스택과 백그라운드, 태스크 큐를 관리
- 백그라운드(Background)는 타이머나 이벤트 리스너와 같은 비동기 함수들이 실행되는 곳
- 태스크 큐(Task Queue)는 비동기 함수의 콜백 함수나 이벤트 리스너의 콜백 함수가 일시적으로 보관되는 곳
- 이벤트 루프는 호출 스택이 비어있으면 태스크 큐에서 함수를 꺼내서 호출 스택에 쌓음

```javascript
function foo() {
  console.log("foo");
}

function bar() {
  console.log("bar");
  foo();
}

function baz() {
  console.log("baz");
  bar();
}

baz();
```

### 1.5.3 태스크 큐와 마이크로 태스크 큐

- 태스크 큐는 비동기 함수의 콜백 함수나 이벤트 리스너의 콜백 함수가 일시적으로 보관되는 곳
- 마이크로 태스크 큐(micro task queue)는 프로미스의 후속 처리 메서드의 콜백 함수가 일시적으로 보관되는 곳
- 마이크로 태스크 큐는 태스크 큐보다 우선순위가 높음
- 마이크로 태스크 큐는 프로미스의 후속 처리 메서드의 콜백 함수가 일시적으로 보관되는 곳
- 마이크로 태스크 큐는 태스크 큐보다 우선순위가 높기 때문에, 태스크 큐에 대기 중인 함수보다 마이크로 태스크 큐에 대기 중인 함수가 먼저 호출됨
- 테스크 큐와 마이크로 태스크 큐에 대기 중인 함수가 모두 있다면, 마이크로 태스크 큐에 대기 중인 함수가 모두 호출된 후에 테스크 큐에 대기 중인 함수가 호출됨
- 라이프 사이클 메서드는 렌더링 결과가 실제 돔에 반영된 후에 호출되는 메서드

> 태스크 큐
> : setTimeout, setInterval, setImmediate
> 마이크로 태스크 큐
> : process.nextTick, Promises, queueMicroTask, MutationObserver

```javascript
function foo() {
  console.log("foo");
}

function bar() {
  console.log("bar");
  setTimeout(foo, 0);
}

function baz() {
  console.log("baz");
  bar();
}

setTimeout(baz, 0);

promise.resolve().then(() => console.log("promise"));
```

<br>
<br>

## 1.6 리액트에서 자주 사용하는 자바스크립트 문법

### 1.6.1 구조 분해 할당

- 구조 분해 할당(destructuring assignment)은 객체나 배열을 해체하여 그 값을 개별 변수에 담을 수 있게 하는 자바스크립트 표현식
- 구조 분해 할당은 비구조화 할당이라고도 함
- 배열의 구조 분해 할당은，의 위치에 따라 값이 결정

```javascript
const object = { a: 1, b: 2 };
const { a, b } = object;
console.log(a); // 1
console.log(b); // 2

const array = [1, 2];
const [one, two] = array;
console.log(one); // 1
console.log(two); // 2
```

```jsx
const key = "a";

const object = {
  a: 1,
  b: 2,
};

const { [key]: one, b: two } = object;

console.log(one); // 1
console.log(two); // 2

const array = [1, 2];

const [one, two] = array;

console.log(one); // 1
console.log(two); // 2
```

### 1.6.2 전개 구문

- 전개 구문(spread syntax)은 객체 혹은 배열을 펼칠 수 있게 해주는 자바스크립트 표현식
- 전개 구문은 펼침 연산자(spread operator)라고도 함

```javascript
const object = { a: 1, b: 2, c: 3 };
const { a, ...c } = object;
console.log(a); // 1
console.log(c); // { b: 2, c: 3 }

const array = [1, 2, 3, 4, 5];
const [one, ...rest] = array;
console.log(one); // 1
console.log(rest); // [2, 3, 4, 5]
```

### 1.6.3 객체 초기자

- 객체 초기자(object initializer)는 객체를 만들 때 변수 이름과 객체의 프로퍼티 이름이 동일하다면 프로퍼티 이름을 생략할 수 있음

```javascript
const x = 0;
const y = 0;

const obj = { x, y };
console.log(obj); // { x: 0, y: 0 }
```

### 1.6.4 Array 프로토타입의 메서드: map, filter, reduce, forEach

- Array 프로토타입의 메서드는 배열을 순회하면서 배열의 각 원소에 대해 특정 작업을 수행할 때 사용
- map 메서드는 배열의 각 원소에 대해 특정 작업을 수행한 후, 그 결과를 새로운 배열에 담아서 반환
- filter 메서드는 배열의 각 원소에 대해 특정 조건을 만족하는 원소만을 따로 추출하여 새로운 배열에 담아서 반환
- reduce 메서드는 배열의 각 원소에 대해 특정 작업을 수행한 후, 그 결과를 새로운 배열에 담아서 반환
- forEach 메서드는 배열의 각 원소에 대해 특정 작업을 수행

```javascript
const array = [1, 2, 3, 4, 5];

const squared = [];
for (let i = 0; i < array.length; i++) {
  squared.push(array[i] * array[i]);
}

console.log(squared); // [1, 4, 9, 16, 25]
```

#### Array.prototype.map

```javascript
const array = [1, 2, 3, 4, 5];

const squared = array.map((n) => n * n);
console.log(squared); // [1, 4, 9, 16, 25]
```

#### Array.prototype.filter

```javascript
const array = [1, 2, 3, 4, 5];

const even = array.filter((n) => n % 2 === 0);
console.log(even); // [2, 4]
```

#### Array.prototype.reduce

```javascript
const array = [1, 2, 3, 4, 5];

const sum = array.reduce((accumulator, current) => accumulator + current, 0);
console.log(sum); // 15
```

#### Array.prototype.forEach

```javascript
const array = [1, 2, 3, 4, 5];

array.forEach((n) => {
  console.log(n);
});
```

### 1.6.5 삼항 조건 연산자

- 삼항 조건 연산자(ternary operator)는 조건문의 축약형
- 삼항 조건 연산자는 조건문의 결과에 따라 반환할 값을 결정

```javascript
const array = [1, 2, 3, 4, 5];

const squared = array.map((n) => (n % 2 === 0 ? n * n : n));
console.log(squared); // [1, 4, 3, 16, 5]
```

<br>
<br>

## 1.7 선택이 아닌 필수, 타입스크립트

### 1.7.1 타입스크립트란?

- 타입스크립트(TypeScript)는 자바스크립트의 슈퍼셋(superset)이자 확장된 언어
- 타입스크립트는 자바스크립트의 모든 기능을 포함하면서 정적 타입(static type)을 지원하는 객체지향 프로그래밍 언어

### 1.7.2 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 활용법

- any 대신 unknown 타입을 사용
  - unknown 타입은 타입을 확정할 수 없는 경우에 사용
  - unknown 타입은 타입을 확정할 수 없기 때문에, 타입을 확정하기 전까지는 다른 타입으로 사용할 수 없음
  - unknown 타입은 타입을 확정하기 전까지는 다른 타입으로 사용할 수 없기 때문에, 타입을 확정하기 전까지는 타입 체크를 강제함
- 타입 가드를 사용

  - instanceof
    ```typescript
    if (e instanceof UnExpectedError) {
      // do something ...
    }
    throw e;
    ```
  - typeof

    ```typescript
    if (typeof value === "string") {
      console.log(value);
    }

    function printAge(age: number | string) {
      if (typeof age === "number") {
        console.log(`나이는 ${age}살 입니다.`);
      } else {
        console.log("나이는 숫자만 입력해주세요.");
      }
    }
    ```

  - in

  ```typescript
  interface Student {
    age: number;
    score: number;
  }

  if ("age" in person) {
    person.age; // person은 Student
    person.score;
  }
  ```

- 타입 단언을 사용
- 제네릭을 사용

```typescript
function getFirstAndLast<T>(list: T[]): [T, T] {
  return [Ust[0], list[list.length - 1]];
}
const [first, last] = getFirstAndLast([l, 2, 3, 4, 5]);
first; // number
last; // number
const [first, last] = getFirstAndLast(["a", "b", "c", "d", "e"]);
first; // string
last; //string
```

```typescript
function Component() {
  // state: string
  const [state, setState] = useState<string>("");
  // ...
}
```

여기서 <string>이 제네릭을 사용한 부분입니다. 이는 다음을 의미합니다:

- state의 타입이 string으로 지정됨
- setState는 자동으로 string 타입의 값만 받을 수 있게 됨

```typescript
function multipleGeneric<First, Last>(al: First, a2: Last): [First, Last] {
  return [al, a2];
}
const [a, b] = multipleGeneric<string, boolean>("true", true);
```

- 인덱스 시그니처
  동적인 객체를 정의할 때 유용하지만，단 키의 범위가 앞선 예제의 경우 string으로
  너무 커지기 때문에 존재하지 않는 키로 접근하면 위와 같이 undefined를 반환

```ts
type Hello = {
  [key: string]: string;
};
```

객체의 키를 좁히기 위해서는 Record, as로 타입 단언하기

```ts
type Hello = Record<"hello" | "hi", string>;
const hello: Hello = {
  hello: "hello",
  hi: "hi",
};
// 타입을 사용한 인덱스 시그니처
type Hello = { [key in "hello" | "hi"]: string };

// Object.keys(hello)를 as로 타입을 단언하는 방법
(Object.keys(hello) as Array<keyof Hello>).map((key) => {
  const value = hello[key];
  return value;
});

//타입 가드 함수를 만드는 방법
function keysOf<T extends Object>(obj: T): Array<keyof T> {
  return Array.from(Object.keys(obj)) as Array<keyof T>;
}

keysOf(hello).map((key) => {
  const value = heUo[key];
  return value;
});
```

> WHY? Object.keys는 string[]을 반환할까?
> 자바스크립트는 다른 언어에 비해 객체가 열려 있는 구조로 만들어져 있으므로 덕 타이핑(duck
> typing)으로 객체를 비교해야 하는 특징이 있다.
> 타입스크립트의 핵심 원칙은 타입 체크를 할 때 그 값이 가진 형태에 집중
> 입스크립트는 이렇게 모든 키가 들어올 수 있는 가능성이 열려
> 있는 객체의 키에 포괄적으로 대응하기 위해 string[]으로 타입을 제공하는 것
> 일부 개발자들은 정확한 타입을 반환하는 Exact라는 새로운 타입을 요청하고 있다

```ts
type Car = { name: string };
type Truck = Car & { power: number };
function horn(car: Car) {
  console.log(`${car.name}O| 경적을 울립니다! 빵빵`);
}
const truck: Truck = {
  name: "비싼차",
  power: 100,
};
// 정상적으로 작동한다
// Car에 필요한 속성은 다 가지고 있기 때문에 Car처럼 name을 가지고 있으므로 유효하다
horn(truck);
```

> [Exact Types](https://github.com/Microsoft/TypeScript/issues/12936)

- 타입스크립트의 유틸리티 타입을 사용

### 1.7.3 타입스크립트 전환 가이드

- tsconfig.json 파일을 생성
- JSDoc 과 @ts-check를 사용 (자바스크립트 파일을 굳이 타입스크립트로 전환하지 않더라도 타입을 체크하는 방법)
- @types 패키지를 설치
