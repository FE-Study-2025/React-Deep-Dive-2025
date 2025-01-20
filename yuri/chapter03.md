# 3장. 리액트 훅 깊게 살펴보기

## 훅이란?
훅은 리액트 16.8 버전에 추가된 기능으로, 클래스 컴포넌트에서 가능했던 state, ref 등 리액트 핵심 기능을 함수에서도 가능하게 해주는 기능이다. 클래스형 컴포넌트보다 간결하게 작성할 수 있고, 컴포넌트 재사용성을 높여준다.

## 1. useState
useState는 함수형 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅이다.
```jsx
import { useState } from 'react';
const [state, setState] = useState(initialState);
```

useState는 두 개의 인수를 받는다. 첫번째 인수는 상태의 초기값이고, 두번째 인수는 상태를 업데이트하는 함수이다.

- 리액트에서 렌더링은 함수형 컴포넌트의 return 값을 통해 이전의 리액트 트리와 비교해 리렌더링이 필요한 부분만 업데이트해 이뤄진다.
- 함수형 컴포넌트는 매번 함수를 실행해 렌더링이 일어나고, 함수 내부의 값은 함수가 실행될 때마다 다시 초기화된다.
- 매번 실행되는 함수형 컴포넌트 환경에서 state의 값을 유지하고 사용하기 위해서 리액트는 클로저를 활용하고 있다. 
    - 클로저를 사용함으로써 외부에 해당 값을 노출시키지 않고 오직 리액트에서만 쓸 수 있었고, 함수형 컴포넌트가 매번 실행되더라도 useState에서 이전의 값을 정확하게 꺼내 쓸 수 있게 됐다.

### 게으른 초기화
useState의 인수로 특정한 값을 넘기는 함수를 인수로 넣어줄 수 있는데, 이 게으른 초기화 함수는 오로지 최초의 state 값을 넣을 때만 사용된다. 이후에 리렌더링 시에는 실행되지 않는다.

useState의 초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하라고 돼 있다.
- localStorage나 sessionStorage에서 데이터를 가져오는 경우
- map, filter, reduce 등의 함수를 사용해 배열을 초기화하는 경우
- 초깃값 계산을 위해 함수 호출이 필요할 때

## 2. useEffect
- 클래스형 컴포넌트의 생명주기 메서드를 대체하기 위해 만들어진 훅이라는 말은 정확하지 않다. 
- useEffect는 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘이다. 
- 이 부수 효과가 '언제' 일어나는지보다 어떤 상태값과 함께 실행되는지 살펴보는 것이 중요하다.

```jsx
function Component() {
  useEffect(effect, [dependencies])
}
```

useEffect는 두 개의 인수를 받는다. 
- 첫번째 인수는 부수 효과를 정의하는 함수이고, 
- 두번째 인수는 부수 효과가 실행되는 조건을 정의하는 의존성 배열이다.

#### useEffect는 어떻게 의존성 배열이 변경될 때마다 첫번째 인수인 콜백이 실행될까? 
의존성에 있는 값, state와 props의 변경에 따라 컴포넌트가 리렌더링되는 것을 활용해 부수 효과를 실행하는 것이다.

### 클린업 함수
- 언마운트라기보다는, 함수형 컴포넌트가 리렌더링됐을 때 useEffect의 콜백에 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다. 
- 이 클린업 함수는 의존성 변화 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념이다.

예를 들어, 컴포넌트가 실행될 때 useEffect 콜백 내 이벤트 리스너를 등록하고, 리렌더링 시 클린업 함수로 이전 이벤트 리스너를 해제한다. 
이렇게 함으로써 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제해 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있다.

### 의존성 배열
useEffect의 두번째 인수인 의존성 배열은 컴포넌트가 렌더링 된 후에 콜백이 실행되는 조건을 정의하는 배열이다.  

```jsx
function Component() {
  useEffect(() => {
    console.log('렌더링됨');
  });
}

function Component() {
    console.log('렌더링됨');
}
```
- 의존성 배열이 없는 useEffect는 매 렌더링마다 실행되는데, useEffect를 쓰지 않고 직접 실행하는 경우와 차이점이 있다.
- useEffect는 컴포넌트 렌더링이 완료된 이후에 실행되고, 직접 실행은 컴포넌트가 렌더링되는 도중에 실행되기 때문에 서버 사이드 렌더링의 경우에 서버에서도 실행된다. 
- 그리고 직접 실행은 함수형 컴포넌트의 반환을 지연시키기 때문에 렌더링을 방해하므로 성능에 악영향을 미칠 수 있다.

### useEffect의 구현
```jsx
function useEffect(effect, dependencies) {
  const hooks = global.hooks;

  let previousDependencies = hooks[index];

  let isDependenciesChanged = previousDependencies 
    ? dependencies.some((value, index) => !Object.is(value, previousDependencies[index])) 
    : true;

  if(isDependenciesChanged) {
    callback();
  }
  hooks[index] = dependencies;
  index++;
}
```
의존성 배열의 이전 값과 현재 값을 Object.is 메서드를 통해 얕은 비교를 수행하고, 값이 변경됐을 때만 콜백으로 선언한 부수 효과를 실행한다.

### useEffect 사용의 주의점
- useEffect에 의존성 배열 인수에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야만 하는지, 혹은 여기서 호출하는 게 최선인지 한 번 더 검토해야 한다.  
    - react-hooks/exhaustive-deps 룰은 useEffect 콜백 내부에서 사용하는 값중 의존성 배열에 포함돼 있지 않은 값이 있을 때 경고를 발생시키는데, eslint-disable-line react-hooks/exhaustive-deps 주석을 달아 무시할 수 있다. 
    ```jsx
    useEffect(() => {
      console.log(props);
    }, []); // eslint-disable-line react-hooks/exhaustive-deps
    ```
    - 빈 배열을 의존성으로 할 때, 즉 컴포넌트를 마운트하는 시점에서만 무언가를 하고 싶다라는 의도로 작성하는 경우가 많은데, 컴포넌트의 state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개로 작동하게 되는 것이다.
- useEffect의 첫번째 인수에 함수명을 부여하라. 
    - 변수에 적절한 이름을 붙이는 이유는 해당 변수가 왜 만들어졌는지 파악히기 위함이다. 
    - 마찬가지로 useEffect도 콜백함수에 적절한 이름을 붙이면 해당 useEffect의 목적을 명확히 할 수 있고 그 책임을 최소한으로 좁힐 수 있다.
- 거대한 useEffect 콜백을 작성하지 말라. 부득이하게 큰 useEffect를 만들어야 한다면 적은 의존성 배열을 사용하는 여러 개의 useEffect로 나눠 작성하라.
- useEffect 내에서 사용할 부수 효과라면 외부 함수를 만들지 말고, 내부에서 만들어서 정의해서 사용하는 편이 가독성이 좋고, 불필요한 의존성 배열을 줄일 수 있다.

## 3. useMemo, useCallback
- useMemo는 비용이 큰 연산에 대한 결과를 메모이제이션해 두고, 이 저장된 값을 반환한다. 
- useCallback은 인수로 넘겨받은 콜백 함수 자체를 메모이제이션해 재사용한다. 함수의 재생성을 막아 불필요한 리소스 또는 리렌더링을 방지하고 싶을 때 사용한다.
- useMemo가 값을 기억했다면, useCallback은 함수를 기억한다.

## 4. useRef
useRef는 렌더링을 발생시키지 않고 원하는 상태값을 저장할 수 있는 특징이 있으며 보통 DOM에 접근할 때 사용한다. 
useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
```jsx
function usePrevious(value) {
    const ref = useRef(value);
    useEffect(() => {
        ref.current = value;
    }, [value]); // value가 변경될 때마다 ref.current를 value로 업데이트
    return ref.current;
}

function Component() {
    const [count, setCount] = useState(0);
    const previousCount = usePrevious(count);

    function handleClick() {
        setCount(prev => prev + 1);
    }
    return <button onClick={handleClick}>{count} {previousCount}</button>;
}
```
개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해 주고 싶다면 useRef를 사용하는 것이 좋다.

## 5. useContext
props drilling을 극복하기 위한 Context API를 함수형 컴포넌트에서 사용할 수 있게 해주는 훅이다.
```jsx
const Context = createContext<{hello: string} | undefined>();

function ParentComponent() {
    const [value, setValue] = useState({hello: 'world'});
    return <Context.Provider value={value}>
        <ChildComponent />
    </Context.Provider>;
}

function ChildComponent() {
    const value = useContext(Context); // useContext는 상위에 선언된 Context.Provider에서 제공한 값을 사용할 수 있게 된다.
    return <div>{value.hello}</div>;
}
```

useContext를 사용할 때 주의할 점
- useContext를 함수형 컴포넌트 내부에서 사용할 때 Provider에 의존성을 가지게 되므로 해당 컴포넌트 재활용이 어려워진다는 점을 염두해야 한다.
- 엄밀히 따지면 context는 상태 관리를 해주는 용도가 아니라 상태를 주입해 주는 용도이다.
    - 상태관리 라이브러리가 되는 최소 두가지 조건 모두 하지 못한다.
        1. 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
        2. 필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.

## 6. useReducer
useReducer는 useState의 심화버전이다. 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.

```tsx
type State = {
    count: number;
};

type Action = {
    type: 'increment' | 'decrement' | 'reset';
    payload: State;
};

function init(initialState: State) {
    return initialState;
}

const initialState: State = {count: 0};

// useReducer의 기본 action을 정의하는 함수. 앞서 선언한 State와 Action을 기반으로 상태를 어떻게 변경할지 정의
function reducer(state: State, action: Action): State {
    switch(action.type) {
        case 'increment':
            return {count: state.count + 1};
        case 'decrement':
            return {count: state.count - 1 > 0 ? state.count - 1 : 0};
        case 'reset':
            return init(action.payload || initialState);
        default:
            throw new Error('Unexpected action type');
    }
}

// useReducer는 컴포넌트 내부에서 사용되며, 초기 상태와 리듀서 함수, 게으른 초기화 함수를 인수로 받는다.
const [state, dispatch] = useReducer(reducer, initialState, init);

function handleUpClick() {
    dispatch({type: 'increment'});
}

function handleDownClick() {
    dispatch({type: 'decrement'});
}

function handleResetClick() {
    dispatch({type: 'reset', payload: {count: 0}});
}
```
- useReducer를 사용하는 목적은 복잡한 형태의 state를 사전에 정의된 dispatch로만 수정할 수 있게 제한하여, state의 변화를 예측하기 쉽게 만드는 것이다.
- 여러개의 state를 관리하는 것보다 성격이 비슷한 여러 개의 state를 묶어 useReducer로 관리하면 state를 사용하는 로직과 이를 관리하는 비즈니스 로직을 분리할 수 있다.
- useReducer나 useState 둘 다 세부 동작과 쓰임에만 차이가 있을 뿐, 결국 클로저를 활용해 state를 관리한다는 점은 동일하다.

## 7. useImperativeHandle
React.forwardRef는 자식 컴포넌트에게 ref 참조를 전달할 때, 예약어로 지정된 ref를 그대로, 일관된 이름으로 전달 할 수 있게 하는 훅이다.
useImperativeHandle은 부모에게서 넘겨받은 ref의 값에 원하는 값이나 액션을 정의할 수 있다.
```jsx
const ChildComponent = forwardRef((props, ref) => {
    useImperativeHandle(ref, () => ({
        focus: () => {
            ref.current.focus();
            console.log('focus');
        }
    }), [props.value]);

    return <input ref={ref} {...props} />;
});
```

## 8. useLayoutEffect
useLayoutEffect는 useEffect와 비슷하지만, 브라우저에 변경 사항을 반영하기 전에 실행된다.
1. react가 DOM을 업데이트
2. useLayoutEffect 실행
3. 브라우저에 변경 사항을 반영
4. useEffect 실행

그리고 useLayoutEffect를 이해하기 위해 중요한 사실은 "모든 DOM 변경 후에 useLayoutEffect의 콜백 함수 실행이 '동기적으로' 실행된다"는 것이다.
'동기적'으로 실행한다는 것은 react는 useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면을 그린다는 것을 의미하고, 이러한 작동방식으로 힌해 성능 문제가 발생할 수 있다.
그래서 DOM은 계산됐지만 이것이 화면에 반영되지 전에 하고 싶은 작업이 있을 때 (DOM요소를 기반으로 한 애니메이션 또는 스크롤 위치 제어 등) useLayoutEffect를 사용하는 것이 좋다.

## 9. 훅의 규칙
- 훅은 함수형 컴포넌트, 사용자 정의 훅 내부에서만 호출해야 한다.
- 훅은 최상위 레벨에서만 호출해야 한다.
- 훅은 조건문, 반복문 내부에서 호출하지 않아야 한다. (최근 리액트 실험 버전에서 use라고 하는 반복문, 조건문 등에서도 실행 가능한 훅이 나왔다.)

```jsx
function Component() {
    const [count, setCount] = useState(0);
    useEffect(() => {
        // do something...
    }, [count]);
}
```
위 컴포넌트는 파이버에서 다음과 같이 저장된다.
```javascript
{
    memoisedState: 0,
    baseState: 0,
    queue: {},
    baseUpdate: null,
    next: { // useEffect 훅
        memoisedState: {
            tag: 192,
            create: () => {},
            destroy: undefined,
            deps: [0, false],
            next: { /** ... */ }
        },
        baseUpdate: null,
        queue: null,
        baseUpdate: null,
    }
}
```
리액트 훅은 파이버 객테의 링크드 리스트의 호출 순서에 따라 저장된다. 
그 이유는 각 훅이 파이버 객체 내에서 순서에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문이다. 
이렇게 고정된 순서에 의존해 훅과 관련된 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해진다.
그러므로 항상 훅은 실행 순서를 보장받을 수 있는 컴포넌트 최상단에 선언돼 있어야 한다.

## 10. 사용자 정의 훅
- 사용자 정의 훅은 내부에 useState, useEffect, useContext, useReducer 등의 리액트 훅을 사용하여 자신만의 원하는 룩으로 만드는 기법이다.
- 사용자 정의 훅 내부에서 이러한 리액트 훅을 사용하고 있기 때문에, 리액트 훅의 이름은 "use"로 시작해야 한다는 규직을 사용자 정의 훅도 준수해야 한다.
    - react-hooks/rules-of-hooks의 도움을 받기 위해서는 훅의 이름을 "use"로 시작해야 한다.

## 11. 고차 컴포넌트(Higher-Order Component, HOC)
- 고차 컴포넌트는 컴포넌트 자체의 로직을 재사용하기 위한 방법이다. 컴포넌트 전체를 감쌀 수 있다는 점에서 사용자 정의 훅보다 더욱 큰 영향력을 컴포넌트에 미칠 수 있다.
- React.memo는 컴포넌트의 렌더링 최적화를 위한 고차 컴포넌트이다.
- 리액트의 고차 컴포넌트 구현시 "with"라는 접두사를 사용하는 것이 관례이다.
- 고차 컴포넌트를 사용할 때 부수 효과를 최소화해야 props가 변경될 지 모른다는 우려를 줄일 수 있다. 만약 컴포넌트에 무언가 추가적인 정보를 제공해 줄 목적이라면 별도 props를 전달해 주는 것이 좋다.
- 여러 개의 고차 컴포넌트가 반복적으로 컴포넌트를 감살 경우 복잡성이 매우 커지므로 최소한으로 사용하는 것이 좋다.

## 12. 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?
#### 사용자 정의 훅이 필요한 경우
- 단순히 useState, useEffect와 같이 리랙트에서 제공하는 훅으로만 공통 로직을 격리할 수 있다면 사용자 정의 훅을 사용하는 것이 좋다.
- 보통 사용자 정의 훅에서 반환하는 값을 제공하면 이에 대한 처리는 컴포넌트를 사용하는 쪽에서 원하는 대로 사용 가능하기 때문에 부수 효과가 비교적 제한적이라고 볼 수 있다.
- 반면 고차 컴포넌트는 대부분 렌더링에 영향을 미치는 로직이 존재하므로 사용자 정의 훅에 비해 예측하기 어렵다.

#### 고차 컴포넌트를 사용해야 하는 경우
```jsx
function HookComponent() {
    const { loggedIn } = useLogin();

    if(!loggedIn) {
        return <LoginComponent />;
    }

    return <>안녕하세요.</>;
}

const HOCComponent = withLoginComponent(() => {
    // 로그인 상태에 따른 제어는 고차 컴포넌트에서 하므로 컴포넌트에 필요한 로직만 추가해서 간단해졌다.
    return <>안녕하세요.</>;
});
```

- login이 false인 경우에 렌더링을 해야하는 컴포넌트는 동일하지만 사용자 정의 훅은 해당 컴포넌트가 반환하는 랜더링 결과물에 까지 영향을 미치기 어렵다.
- 이러한 중복 처리가 해당 사용자 정의 훅을 사용하는 애플리케이션 전반에 걸쳐 나타나게 될 것이므로 사용자 정의 훅보다는 고차 컴포넌트를 사용하는 것이 좋다.
- 함수형 컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용하자.




