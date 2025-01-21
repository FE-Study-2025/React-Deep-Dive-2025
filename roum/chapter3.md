keywords
- 랙시컬 환경
- 스코프
- 클로저
- 스냅샷
- 태스크큐

### 1. 클로저 관점에서 리액트

```
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}

```

- 클로저와 var의 관계
1) setTimeout 콜백 함수는 클로저로 동작
- 클로저는 자신이 선언된 렉시컬 환경(스코프)을 기억
- 여기서 각 콜백 함수는 for 루프의 전역 스코프에 있는 i 변수를 기억
2) 그러나 var로 선언된 i는 공유 변수로, for 루프가 끝난 후에도 계속 변경됩
- 따라서 각 콜백 함수는 실행될 때 **최종 값(5)**을 참조

https://ui.dev/javascript-visualizer


### 클로저와 메모리
: 클로저는 함수와 함께 해당 함수가 선언될 때의 렉시컬 환경을 캡처 -> 클로저가 참조하는 모든 변수는 메모리에 계속 유지
클로저를 과도하게 사용하거나 불필요하게 생성하면 메모리 누수(memory leak)나 과도한 메모리 사용으로 성능이 저하될 가능성이 있음

-> 리액트의 최적화 메커니즘으로 브라우저에 나쁜 영향을 주지 않는다.
e.g. Virtual DOM, 컴포넌트 재사용, hooks의 의존성 배열

### 비동기 통신의 이해

keywords
- 프로세스
- 스레드
- Run-to-completion : 자바스크립트의 모든 코드는 동기식으로 한 번에 하나씩 순차적으로 실행

자바스크립트는 싱글 스레드임에도 비동기 통신이 가능함 -> how?

#### 이벤트 루프
: 이벤트루프는 ECMAScript 자바스크립트 표준이 아니다. 
자바스크립트 런타임 외부에서 자바스크립트 비동기 실행을 돕기 위해 만들어진 장치이다. 

이벤트 루프가 하는 일
1) 호출 스택이 비어있는지 여부를 확인한다.
2) 수행해야 하는 코드가 있다면 자바스크립트 엔진을 이용해 실행함


```
function bar() {
    console.log('bar');
}

function baz() {
    console.log('baz');
}

function foo() {
    console.log('foo');
    setTimeout(bar(), 0);
    baz();
}

foo();

```

setTimeout은 실행스택에서 제거되고, 바로 태스크큐에 들어간다.
이벤트 루프가 호출 스택이 비워져 있다는 것을 확인한 뒤 태스크 큐를 실행 스택에 들여보낸다.

-> setTimeout은 정확하게 0초뒤에 실행되지 못한다.

c.f. 태스크 큐 : 실행해야 할 태스크의 집합. 자료 구조 큐가 아닌 Set과 비슷한 동작 방식. (중복 태스크 제거)


#### 비동기 작업은 자바스크립트의 메인 스레드가 아닌, 태스크 큐가 할당되는 별도의 스레드에서 수행된다.
- JavaScript 엔진은 단일 스레드에서 실행되며, 동시에 여러 작업을 처리할 수 없다.
- 비동기 작업은 태스크 큐(Task Queue)에서 관리되며, 이벤트 루프(Event Loop)를 통해 콜 스택으로 전달

* setTimeout 자체는 JavaScript 엔진(V8 같은 엔진)에서 실행되지 않고, setTimeout은 브라우저(Web API) 또는 Node.js의 백그라운드 환경에서 관리됨
* setTimeout은 자바스크립트 언어 자체의 기능이 아니라 브라우저 환경에서 제공하는 Web API 기능

![alt text](https://velog.velcdn.com/images/onedanbee/post/a0e56831-c663-4953-b892-8dd76cee2159/image.gif)
![alt text](https://velog.velcdn.com/images/onedanbee/post/529ec18f-abf2-4fc0-88e2-86c2cc4fe3ce/image.gif)


reference: https://velog.io/@onedanbee/%EB%B9%84%EB%8F%99%EA%B8%B0-Task-Queue-Macrotask-Queue-%EC%99%80-Microtask-Queue-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C



c.f. 마이크로 태스크 큐 : 기존 태스크 큐보다 우선권을 가짐
 -Promise


----


## 2. 게으른 초기화(lazy initialization)

: useState에 변수 대신 함수를 넘기는 것

> state가 처음 만들어질 때만 사용된다. 만약 이후에 리렌더링이 발생된다면 이 함수의 실행은 무시된다.


### 왜 이런 현상이 일어날까?

```
const [state, setState] = useState(expensiveCalculation());
```
1) useState가 호출되기 전에 이미 expensiveCalculation()는 실행이 된다.
2) 따라서, useState에는 함수가 실행된 결과값이 전달된다.
3) 렌더링이 발생할 때마다 expensiveCalculation()이 항상 실행되므로, 불필요한 연산이 발생한다.
4) 실행된 결과값은 useState의 초기화에 더 이상 사용되지 않는다. (불필요 렌더링)


```
const [state, setState] = useState(() => expensiveCalculation());
```
1) useState에 함수 () => expensiveCalculation() 자체를 전달
2) 리액트는 전달된 함수의 참조를 내부적으로 저장한다.
3) **자바스크립트**에서는 함수가 실행되지 않고 참조로 전달되면, 해당 함수는 필요할 때만 실행



### 리액트의 상태 관리 구조
리액트는 상태를 효율적으로 관리하기 위해 컴포넌트별로 상태 값을 메모리에 저장한다.
그리고, 최초 렌더링과 리렌더링에 다른 저장 방식을 사용한다.

1) 최초 렌더링: 
- useState에 전달된 함수가 실행되고, 그 결과가 상태 값으로 저장된다.
- 상태 값이 리액트의 컴포넌트 상태 트리에 기록된다.

2) 리렌더링:
- 리액트는 이미 저장된 상태 값을 재사용한다.
- 그래서 useState에 전달된 초기화 함수는 호출되지 않는다. (이미 설정된 상태 값을 유지하고, 전달된 초기값이나 초기화 함수는 무시)


### 예시
```
function MyComponent() {
  console.log("Component rendering...");
  
  const [state, setState] = useState(expensiveCalculation());

  return <div>{state}</div>;
}

function expensiveCalculation() {
  console.log("Expensive calculation executed");
  return 42;
}

```

MyComponent가 리렌더링될 때, expensiveCalculation()이 다시 호출
하지만 useState는 이미 초기화된 상태 값을 유지하므로, expensiveCalculation()의 실행 결과는 무시
즉, 실행된 값은 상태 초기화에 사용되지 않음
state는 여전히 이전의 초기화 값(42)를 유지


### 리액트의 최적화 설계
: 리액트는 성능 최적화를 위해, 상태 초기화 함수가 최소한의 필요 조건에서만 실행되도록 설계
=> useState는 초기화 함수가 최초 렌더링에서만 실행되도록 보장
