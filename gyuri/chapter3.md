# 03 리액트 훅 깊게 살펴보기

훅 (hook) : 클래스형 컴포넌트의 생명주기 메서드를 대체하는 등의 다양한 작업

훅을 왜 사용하는데?

- 클래스형 컴포넌트에서만 가능했던 state, ref 등 리액트의 핵심적인 기능을 함수에서도 가능하게 만들기 위해
- 클래스형 컴포넌트보다 간결하게 작성할 수 있어서

### useState

- 초기값: 인수로 넘겨준다. 없을 경우 `undefined`
- 반환값: 배열
  - 첫번째 원소: state 값 자체
  - 두번째 원소: setState 함수, state 값을 변경 할 수 있다.
  - [setState](https://kyounghwan01.github.io/blog/React/React18/flushsync/#%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%A9%E1%86%AF-%E1%84%8B%E1%85%A8%E1%84%89%E1%85%B5)

```typescript
const [state, setstate] = useState(initialState);
```

Q. `let` 으로 변수를 정의해서 사용하면 안될까요?

```typescript
function Component() {
  const [, triggerRender] = useState();

  let state = "hello";

  function handleButtonClick() {
    state = "hi";
    triggerRender();
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  );
}
```

A. ❌ NO!

- 리액트의 렌더링은 함수형 컴포넌트에서 반환한 결과물인 return의 값을 비교해 실행된다.
- 매번 렌더링이 발생될 때마다 함수는 다시 새롭게 실행되고 새롭게 실행되는 함수에서 state는 매번 hello로 초기화된다.

> ⚒️ 함수형 컴포넌트는 매번 함수를 실행해 렌더링이 일어나고，함수 내부의 값은 함수가 실행될 때마다 다시 초기화된다

Q. 그러면 `useState` 는 어떻게 랜더링이 일어나도 값을 유지할 수 있을까?

A. `클로저`를 사용했기 때문 (예제 코드 3.16 P.193)

- 클로저는 어떤 함수(useState) 내부에 선언된 함수 (setstate)가 함수의 실행이 종료된 이후에도(useState가 호출된 이후에도) 지역변수인 state를 계속 참조할 수 있다는 것을 의미
- 클로저를 사용함으로써 외부에 해당 값을 노출시키지 않고 오직 리액트에서만 쓸 수 있었고，함수형 컴포넌트가 매번 실행되더라도 useState에서 이전의 값을 정확하게 꺼내 쓸 수 있게 됐다.

참고) 리액트는 깃허브 저장소에서 훅에 대한 구현체를 타고 올라가다 보면 접근을 못하게 막아두었다. 따라서 정확한 구현을 알기 어렵고 Preact에는 되어있다. 리액트보다 가볍고 모든 코드를 명확하게 볼 수 있다.

#### 게으른 초기화

- useState에 변수 대신 함수를 넘기는 것을 게으른 초기화(lazy initialization)라고 한다
- 게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용된다. 이후에 리렌더링이 발생된다면 이 함수의 실행은 무시된다.

```typescript
// 일반적인 useState 사용
// 바로 값을 집어넣는다.
const [count, setCount] = useState(
  Number.parselnt(window.localstorage.getltem(cacheKey))
);

// 게으른 초기화
// 위 코드와의 차이점은 함수를 실행해 값을 반환한다는 것이다.
const [count, setCount] = useState(() =>
  Number.parselnt(window.localstorage.getltem(cacheKey))
);
```

1. 초깃값이 필요한 최초 렌더링과, 초깃값이 있어 더 이상 필요 없는 리렌더링 시에도 동일하게 계속 해당 값에 접근해서 낭비가 발생한다.
2. 이런 경우에는 함수 형태로 인수에 넘겨주는 편이 훨씬 경제적
3. 초깃값이 없다면 함수를 실행해 무거운 연산을 시도할 것이고，이미 초깃값이 존재한다면 함수 실행을 하지 않고 기존 값을 사용할 것 이다.

Preact에서 useState 구현 [링크](https://github.com/preactjs/preact/blob/d3d57db6ece98c5a3bbaa91777a4942f155935f6/hooks/src/index.js#L173)

### useEffect

- useEffect는 두 개의 인수를 받는데, 첫 번째는 콜백, 두 번째는 의존성 배열이다. 두 번째 의존성 배열의 값이 변경되면 첫 번째 인수인 콜백을 실행한다.
- useEffect는 클린업 함수를 반환할 수 있는데, 이 클린업 함수는 컴포넌트가 [리랜더링](https://react.dev/reference/react/useEffect#my-cleanup-logic-runs-even-though-my-component-didnt-unmount)될 때 실행된다.

Q. useEffect는 어떻게 의존성 배열이 변경된 것을 알고 실행될까?

A.

- 함수형 컴포넌트는 매번 함수를 실행해 렌더링을 수행한다
- 렌더링할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있으 면 부수 효과를 실행
- 자바스크립트의 proxy나 데이터 바인딩，옵저버 같은 특별한 기능을 통해 값의 변화를 관찰하 는 것이 아니고 하는 평범한 함수라 볼 수 있다
- Preact에서 useEffect 구현 [링크](https://github.com/preactjs/preact/blob/d3d57db6ece98c5a3bbaa91777a4942f155935f6/hooks/src/index.js#L288)

#### 클린업 함수

- 일반적으로 이 클린업 함수는 이벤트를 등록하고 지울 때 사용
- 새로운 값과 함께 렌더링된 뒤에 실행된다
- ex) 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는 것이다. 이렇게 함으로써 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있다.

함수형 컴포넌트의 특징 때문이라고 생각할 수 있다.
렌더링 될 때 마다 함수 전체를 다시 실행하기 때문에, 이런 클린업 함수를 통해서 이전 랜더링때 사용했던 이벤트 핸들러를 없애줘야하는 것!

#### 의존성 배열

- 아무런 값도 넘겨주지 않는다면 이때는 의존성을 비교할 필요 없이 렌더링할 때마다 실행이 필요하다고 판단해 렌더링이 발생할 때마다 실행된다.

Q. 의존성 배열이 없는 useEffect가 매 렌더링마다 실행된다면 그냥 useEffect 없이 써도 되는 게 아닐까?

```typescript
// 1

function Component() {
  console.log("렌더링됨");
}

// 2

function Component() {
  useEffect(() => {
    console.log("렌더링됨");
  });
}
```

A. 서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해 준다.

useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 된다.

useEffect는 컴포넌트 렌더링의 부수 효과, 즉 컴포넌트의 렌더링이 완료된 이후에 실행된다. 반면 직접 실행은 컴포넌트 가 렌더링되는 도중에 실행된다. 따라서 1번과는 달리 서버 사이드 렌더링의 경우에 서버에서도 실행된다. 그리고 이 작업 은 함수형 컴포넌트의 반환을 지연시키는 행위다. 즉, 무거운 작업일 경우 렌더링을 방해하므로 성능에 악영향을 미칠 수 있다.

#### useEffect를 사용할 때 주의할 점

1. eslint-disable시ine react-hooks/exhaustive-deps 주석은 최대한 자제하라
   1. 빈 배열 []을 의존성으로 할 때，즉 컴표넌트를 마운트하는 시점 에만 무언가를 하고 싶을때
   2. WHY? 이는 클래스형 컴포넌트의 생명주기 메서드인 componentDidMount에 기반한 접근법으로，가급적이면 사용해선 안 된다
   3. 의존성 배열 을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 컴표넌트의 state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개로 작동하게 된다는 것이다
   4. useEffect에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야 만 하는지，혹은 여기서 호출하는 게 최선인지 한 번 더 검토해 봐야 한다.
   5. 실행 위치를 다시 한번 고민해 보는 것이 좋다.
2. useEffect의 첫 번째 인수에 함수명을 부여하라
   1. 코드가 복잡하고 많아질수록 무슨 일을 하는 useEffect 코드인지 파악하기 어려워진다. useEffect의 인수를 익명 함수가 아닌 적절한 이름을 사용한 기명 함수로 바꾸는 것이 좋다.
3. 거대한 useEffect를 만들지 마라
   1. 랜더링 시 의존성이 변경될 때마다 부수 효과를 실행한다. 이 부수 효 과의 크기가 커질수록 애플리케이션 성능에 악영향을 미친다. 만약 의존성 배열에 불가피하게 여러 변수가 들어가야 하는 상황이라면 최대한 useCallback과 useMemo 등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋다.
4. 불필요한 외부 함수를 만들지 마라
   1. 예를들어 data fetch 하는 함수의 경우, useEffect 내에서 사용할 부수 효과이기에 내부에서 만들어서 정의해서 사용하는 편이 훨씬 도움이 된다. (P.207)

Q. 왜 useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?
A. useEffect에서 비동기로 함수를 호출할 경우 경쟁 상태가 발생할 수 있기 때문이다.
useEffect 내부에서 비동기 함수를 선언해 실행하거나. 즉시 실행 비동기 함수를 만들어서 사용하는 것은 가능하다.

ex)
비동기 함수의 응답 속도에 따라 결과가 이상하게 나타날 수 있다. 극단적인 예제로 이전 state 기반의 응답이 10초 가 걸렸고、이후 바뀐 state 기반의 응답이 1초 뒤에 왔다면 이전 state 기반으로 결과가 나와버리는 불상사가 생길 수 있다.

```typescript
useEffect(() => {
let shouldlgnore = false

async function fetchData() {
const response = await fetch('http://some.data.com') const result = await response.json()
if (!shouldlgnore) {

setData(result) }

}

fetchDataO

return () => {
// shouUilgnore를 이용해 useState의 두 번째 인수를 실행을 막는 것뿐만 아니라
// AbortControUer를 활용해 직전 요청 자체를 취소하는 것도 좋은 방법이 될 수 있다.
shouldlgnore = true
}
}, [])
```

### useMemo

- 큰 연산에 대한 결과를 저장(메모이제이션)해 두고，이 저장된 값을 반환하는 훅
- '값'을 기억해둔다.
- useMemo로 컴포넌트도 감쌀수 있지만, React.memo를 쓰는 것이 더 현명하다.

### useCallback

[codesandbox](https://codesandbox.io/p/devbox/objective-stonebraker-xvwr9q?file=%2Fsrc%2FApp.tsx%3A3%2C1-37%2C2)

- 인수로 넘겨받은 콜백 자체를 기억한다
- 특정 함수를 새로 만들지 않고 다시 재사용한다

```typescript
import { useState, useEffect } from "preact/hooks";

import { memo } from "preact/compat";

interface ChildComponentProps {
  name: string;
  value: boolean;
  onChange: () => void;
}

const ChildComponent = memo(
  ({ name, value, onChange }: ChildComponentProps) => {
    // 렌더링이 수행되는지 확인하기 위해 넣었다.
    useEffect(() => {
      console.log("rendering!", name);
    });

    return (
      <>
        <h1>
          {name} {value ? "켜짐" : "꺼짐"}{" "}
        </h1>
        <button onClick={onChange}>toggle</button>{" "}
      </>
    );
  }
);

export function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);

  const toggle1 = () => {
    setStatus1(!status1);
  };

  const toggle2 = () => {
    setStatus2(!status2);
  };

  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}
```

### useRef

- useRef는 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다. (useState와 동일)
- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.
- useRef의 가장 일반적인 사용 예는 바로 DOM에 접근하고 싶을 때일 것이다.
- useRef를 사용할 수 있는 유용한 경우는 렌더링을 발생시키지 않고 원하는 상태값을 저장할 수 있다는 특징 을 활용해 useState의 이전 값을 저장하는 usePrevious() 같은 흑을 구현할 때다.
- Preact에서의 useRef는 useMemo로 구현되어있다.
- Preact에서 useRef 구현 [링크](https://github.com/preactjs/preact/blob/d3d57db6ece98c5a3bbaa91777a4942f155935f6/hooks/src/index.js#L316)

### useContext

- 부모 컴포넌트와 자식 컴포넌트가 props로 데이터를 넘겨주는 것이 일반적이다.
- 컴포넌트의 거리가 멀어질수록 코드는 복잡해지기 때문에 useContext를 사용한다.
  - prop 내려주기(props drilling)을 방지

```typescript
const Context = createContext<{ hello: string } | undefined>()

function ParentComponent() { return (

<>
<Context.Provider value={{ hello: 'react' }}>

<Context.Provider value={{ hello: 'javascript' }}> <ChildComponent />

</Context.Provider>
</Context.Provider>

</>

) }

function ChildComponentO {
const value = useContext(Context)

// react가 아닌 javascript가 반환된다.

return <>{value ? value.hello : ''}</> }
```

#### useContext를 사용할 때 주의할 점

- 항상 컴포넌트 재활용이 어려워진다는 점을 염두에 둬야 한다
- useContext를 상태 관리를 위한 리액트의 API가 아니다
  - 상태를 주입해 주는 API
- props 값을 하위로 전달해 줄 뿐，useContext를 사 용한다고 해서 렌더링이 최적화되지는 않는다

### useReducer

- useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.

반환값은 useState와 동일하게 길이가 2인 배열이다.

- state: 현재 useReducer가 가지고 있는 값을 의미한다. useState와 마찬가지로 배열을 반환하는데, 동일하게 첫 번 째 요소가 이 값이다
- dispatcher: state를 업데이트하는 함수. useReducer가 반환하는 배열의 두 번째 요소다. setstate는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다는 점이 다르다. 이 action은 state# 변경할 수 있는 액션을 의미한다.
- state 값에 대한 접근은 컴포넌 트에서만 가능하게 하고, 이를 업데이트하는 방법에 대한 상세 정의는 컴포넌트 밖에다 둔 다음, state의 업 데이트를 미리 정의해 둔 dispatcher로만 제한하는 것이다
- 성격이 비슷한 여러 개의 state를 묶어 useReducer로 관리하는 편 이 더 효율적이다.
- Preact 에서는 useState를 useReducer로 만들었다. [링크](https://github.com/preactjs/preact/blob/d3d57db6ece98c5a3bbaa91777a4942f155935f6/hooks/src/index.js#L186C17-L186C27)

### uselmperativeHandle

- 사용 빈도 수는 작지만 React.forwardRef 에서 유용하게 사용될 수 있다

#### forwardRef

- ref를 상위 컴포넌트에서 하 위 컴포넌트로 전달하고 싶다면?
  - forwardRef를 사용한다.
- `parentRef` 같은 네이밍으로 props를 넘길 수 있지만 ref를 전달하는 일관성을 제공하기 위해 사용한다.

- uselmperativeHandle은, 부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다

```typescript
const Input = forwardRef((props, ref) => {
// uselmperativeHandle을 사용하면 ref의 동작을 추가로 정의할 수 있다.
useImperativeHandle(
ref,
0 => ({
alert: () => alert(props.value),
}),
// useEffect의 deps와 같다.
[props.value],
)

return <input ref={ref} {...props} /> })

```

이런식으로

- uselmperativeHandle 흑을 사용해 추가적인 동작을 정의
- 이로 써 부모는 단순히 HTMLElement뿐만 아니라 자식 컴포넌트에서 새롭게 설정한 객체의 키와 값에 대해서도 접근할 수 있다.

### useLayoutEffect

- useEffect와 동일한 모습으로 작동하는 것처럼 보인다.
- 모든 DOM의 변경 후에 useLayoutEffect의 콜백 함수 실행이 동기적으로 발생 (=리액트는useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면 을 그린다는 것을 의미)
- useLayoutEffect가 useEffeet보다는 먼저 실행된다

  실행 순서

1. 리액트가 DOM을 업데이트(렌더링)
2. useLayoutEffect를실행
3. 브라우저에 변경 사항을 반영
4. useEffect를실행

Q. useLayoutEffect는 언제 사용하면 좋을까?
A. DOM은 계산됐지만 이것 이 화면에 반영되기 전에 하고 싶은 작업이 있을 때

DOM 요소를 기반으로 한 애니메이션, 스크롤 위치를 제어하는 등 화면에 반영되기 전에 하고 싶은 작업에 useLayoutEffect를 사용한다면 useEffect를 사용했을 때보다 훨씬 더 자연스러운 사용자 경험을 제공할 수 있다.

라고하는데 아직 경험해본적이 없다.

### useDebugValue

- 디버깅하고 싶은 정보를 이 흑에다 사용하면 리액트 개발자 도구에서 볼 수 있다
- 두 번째 인수로 포매팅 함수를 전달하면 이에 대한 값이 변경됐을 때만 호출되어 포매팅된 값을 노출한다
- 오직 다른 흑 내부에서만 실행할 수 있다

```typescript
function useDateO {
const date = new Date()
// useDebugValue로 디버김 정보를 기록
useDebugValue(date, (date) => '현재 시간: ${date.toISOString()}') return date
}
```

### 훅의 규칙

- ESLint : `react-hooks/rules-of-hooks`

1. 최상위에서만 흑을 호출해야 한다. 반복문이나 조건문. 중첩된 함수 내에서 흑을 실행할 수 없다. 이 규칙을 따라야만 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 혹이 호출되는 것을 보장할 수 있다.
2. 훅을 호출할 수 있는 것은 리액트 함수형 컴포넌트, 혹은 사용자 정의 흑의 두 가지 경우 뿐이다. 일반 자바스크립트 함수에서는 혹을 사용할 수 없다.

- WHY? 고정된 순서에 의존해 흑과 관련된 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해진다

## 사용자 정의 훅과 고차 컴포넌트

### 사용자 정의 훅

- 이름이 반드시 use 로 시작하는 함수
- 복잡하고 반복되는 로직은 사용자 정의 흑으로 간단하게 만들 수 있다
- 리액트 커뮤니티에 다양하게 존재한다!
  [react-use](https://github.com/streamich/react-use)
  [use- Hooks](https://github.com/uidotdev/usehooks)
  [ahooks](https://github.com/alibaba/hooks)

```typescript
function useFetch<T>(
url: string,
{ method, body }: { method: string; body?: XMLHttpRequestBodylnit }, ){

// 응답 결과
const [result, setResult] = useState<T | undefined>()
// 요청 중 여부
const [isLoading, setlsLoading] = useState<boolean>(false)
// 2xx 3xx로 정상 응답인지 여부
const [ok, setOk] = useState<boolean | undefined>()

```

### 고차 컴포넌트

- 고차 컴포넌트(HOC, Higher Order Component)는 컴포넌트 자체의 로직을 재사용하기 위한 방법
- 리액트가 아니더라도 자바스크립트 환경에서 널리 쓰일 수 있다
- 리액트에서 가 장 유명한 고차 컴포넌트는 리액트에서 제공하는 API 중 하나인 React .memo이다.

#### React.memo

- 부모 컴포넌트가 새롭게 렌더링될 때, 자식 컴포넌트의 props 변경 여부와 관계없이 랜더링이 발생한다
- React.memo는 렌더링하기에 앞서 props를 비교해 이전과 props가 같다면 렌더링 자체 를 생략하고 이전에 기억해 둔(memoization) 컴포넌트를 반환한다 (클래스형 컴 포넌트에서 소개했던 PureComp아)ent와 매우 유사)
- useMemo() 로도 메모이제이션 가능하나 혼선 을 빚을 수 있으므로 목적과 용도가 뚜렷한 memo를 사용하는 편이 좋다.

#### 고차 함수

- 고차 함수란? 사전적인 정의를 살펴보면 ‘함수를 인수로 받거나 결과로 반환하는 함수

```typescript
// Array.prototype.map
const doubledList = list.map((item) => item * 2);
```

- useState의 setstate 또한 함수를 결과로 반환하는 조건을 만족하므로 고차 함수라고 할 수 있다.

#### 고차 함수를 활용한 리액트 고차 컴포넌트

- with로 시작하는 이름을 사용해야 한다 (리액트 라우터의 withRouter)
- 고차 컴포넌트를 사용할 때 주의할 점 중 하나는 부수 효과를 최소화해야 한다는 것이다.
- 반드시 컴포넌트의 props를 임의로 수정，추가，삭제하는 일은 없어야 한다
- 복잡성이 커지는 것을 유의한다 (삼중, 사중 컴포넌트를 감싸는 구조)

```typescript

function withLoginComponent<T〉(Component: ComponentType<T>) { return function (props: T & LoginProps) {

const { LoginRequired, . . . restProps } = props
if (loginRequired) {
return <>로그인이 필요합니다.</>
}
return <Component {...(restProps as T)} /> }

}

// 원래 구현하고자 하는 컴포넌트를 만들고, withLoginComponent로 감싸기만 하면 끝이다.
// 로그인 여부, 로그인이 안 되면 다른 컴포넌트를 렌더링하는 책임은 모두
// 고차 컴포넌트인 withLoginComponent에 맡길 수 있어 매우 편리하다.

const Component = withLoginComponent((props: { value: string }) => {
return <h3>{props.value}</h3> })

```

Q. 서버나 NGINX와 같이 자바스크립트 이전 단계에서 처리하는 편이 훨씬 효율적이다.
어떻게?

#### 사용자 정의 훅 vs 고차 컴포넌트

##### 사용자 정의 훅이 필요한 경우

- useEffect, useState와 같이 리액트에서 제공하는 흑으로만 공통 로직을 격리할수 있다면 사용자 정의 흑을 사용하는 것이 좋다
- 렌더링에 영향을 미치지 못하기 때문에 사용이 제한적이므로 반환하는 값을 바탕으로 무엇을 할지는 개발자에게 달려 있다. 따라서 컴포넌트 내부에 미치는 영향을 최소화해 개발자가 흑을 원하는 방향으로만 사용할 수 있다는 장점이 있다

##### 고차 컴포넌트를 사용해야 하는 경우

- 고차 컴포넌트는 공통화된 렌더링 로직을 처리하기에 좋다
- 사용자 정의 흑은 해당 컴포넌트가 반환하는 랜더링 결과물에까지 영향을 미치기는 어렵기 때문이다
