# 3. 리액트 훅 깊게 살펴보기

## 3.1 리액트의 모든 훅 파헤치기

### useState

- 상태를 정의하고, 관리할 수 있게 해주는 훅 (클로저를 이용)

useState 내부의 모습을 구현한 모습
```javascript
const MyReact = (function(){
    const global = {}
    let index = 0

    function useState(initialState){
        if(!global.states){
            // 애플리케이션 전체 states 배열 초기화. 최초 접근이면 빈 배열로 초기화
            global.states = []
        }

        // states 정보를 조회해서 현재 상태값 있는지 확인, 없으면 초깃값으로 설정
        const currentState = global.states[index] || initialState
        global.states[index] = currentState

        const setState = (function(){
            // 현재 index를 클로저로 가둬놔서 이후에도
            // 계속해서 동일한 index에 접근할 수 있도록 한다.
            let currentIndex = index
            return function(value){
                global.states[currentIndex] = value
                // 컴포넌트 렌더링(코드 생략)
            }
        })()

        // useState를 쓸 때 마다 index를 하나씩 추가한다.
        // index는 setState에서 사용된다.
        // 즉 하나의 state마다 index가 할당되어 있어 그 index가 배열의 값(global.states)을 가리킨다.

        index = index + 1

        return [currentState, setState]
    }
})
```

#### 게으른 초기화 : useState에 변수 대신 함수를 넘기는 것
localStorage나 sessionStorage에 대한 접근, map, filter, find 가튼 배열에 대한 접근, 혹은 초깃값 계산을 위해 함수 호출이 필요할 때와 같이 무거운 연산을 포함해 실행 비용이 많이 드는 경우에 사용하는 것이 좋다.

최초 렌더링, 리렌더링 시에 재호출하는 낭비를 줄일 수 있다.

### useEffect
어플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘

함수형 컴포넌트는 렌더링 시마다 고유의 state와 props 값을 가진다. useEffect는 렌더링할 때마다 의존성에 있는 값을 보면서 의존성이 있는 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행한다.

클린업 함수 : 함수형 컴포넌트가 리렌더링 됐을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념

#### 의존성 배열
- 빈 비열 : 비교할 의존성이 없다고 판단, 최초 렌더링 직후 실행된 다음부터 더 이상 실행되지 x
- 아무런 값 넘겨주지 않았을 때: 의존성 비교 필요 없이 렌더링할 때마다 실행이 필요하다고 판단해 렌더링이 발생할 때마다 실행

useEffect의 구현
```javascript
const MyReact = (function(){
  const global = {}
  let index = 0

  function useEffect(callback, dependencies){
    const hooks = global.hooks

    // 이전 훅 정보가 있는지 확인한다.
    let previousDependencies = hooks[index]

    // 변경됐는지 확인한다.
    // 이전 값이 있다면 이전 값을 얕은 비교로 비교해 변경이 일어났는지 확인한다.
    // 이전 값이 없다면 최초 실행이므로 변경이 일어난 것으로 간주해 실행을 유도한다.
    let isDependenciesChanged = previousDependencies
      ? dependencies.some(
        (value, idx) => !Object.is(value, previousDependencies[idx]).
      )
      : true

    // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행한다.
    if(isDependenciesChanged){
      callback()

      // 다음 훅이 일어날 때를 대비하기 위해 index 추가
      index ++

      // 현재 의존성을 훅에 다시 저장한다.
      hooks[index] = dependencies
    }
  }

  return { useEffect }
})()
```

#### useEffect 사용시 주의할 점
- eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라
- useEffect의 첫 번째 인수에 함수명을 부여하라 - 목적을 파악하기 쉬워진다
- 거대한 useEffect를 만들지 마라 - 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리 / 최대한 useCallback, useMemo 등으로 사전에 정제한 내용들만 useEffect에 담자
- 불필요한 외부 함수를 만들지 마라 - 내부에 만들면 더 간결해진다

#### 왜 useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?
타이밍 문제로 이전 state 기반으로 결과가 나와버리는 등 기대와 다른 결과를 얻을 수 있다. (useEffect의 경쟁 상태 race condition)

인수로 비동기 함수를 지정할 수 없을 뿐, 실행은 할 수 있다.

비동기 함수가 내부에 존재하게 되면 useEffect 내부에서 비동기 함수가 생성되고 실행되는 것을 반복하므로 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가하는 것이 좋다.
fetch의 경우 abortController 등으로 이전 요청을 취소하는 것이 좋다.

```javascript
function Component({ id }: { id: string }) {
    const [info, setInfo] = useState<number | null>(null);
    
    useEffect(() => {
        const controller = new AbortController();

        (async () => {
            const result = await fetchInfo(id, { signal: controller.signal });
            setInfo(await result.json())
        })()
        
        return () => controller.abort();
    }, [id])
    
    return <div></div>
}
```

#### AbortController
웹 요청을 취소할 수 있게 해주는 기능
```javascript
const controller = new AbortController();

setTimeout(() => {
  controller.abort();
}, 2_000);

const URL = 'https://httpbin.org/delay/5';
fetch(URL, { signal: controller.signal })
  .then((res) => {
    console.log(`Received: ${res.status}`);
  }).catch((err) => {
    if (err.name === 'AbortError') {
      console.error('Aborted: ', err);
      return;
    }
    throw err;
  });
```

### useMemo
비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅

### useCallback
useMemo가 값을 기억했다면 useCallback은 인수로 넘겨받은 콜백 자체를 기억

쉽게 말해 특정 함수를 새로 만들지 않고 재사용한다는 의미

### useRef
- useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장
- useRef는 반환값이 객체 내부에 있는 current로 값에 접근 또는 변경 가능
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 x

### useContext
Context를 함수형 컴포넌트에서 사용할 수 있게 해준 useContext 훅

#### 사용 시 주의할 점
- Provider에 의존성을 가지게 되어 컴포넌트 재활용이 어려워짐
- useContext를 사용하는 컴포넌트를 최대한 작게 하거나 재사용되지 않을 만한 컴포넌트에서 사용해야 함
- 컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 함
- 렌더링이 최적화되지는 x

### useReducer
useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리 가능

사용 목적: state 값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는 것

```javascript
type State = {
  count : number
}

// state의 변화를 발생시킬 action의 타입과 넘겨줄 값(payload) 정의
// 꼭 type과 payload라는 네이밍 지킬 필요 x, 객체일 필요 x
// 다만 이러한 네이밍이 가장 널리 쓰임
type Action = { type: 'up' | 'down' | 'reset'; payload?: State }

// count를 받아 초깃값을 어떻게 정의할지 연산
function init(count: State) : State {
  return count
}

const initialState : State = { count : 0 }

// state, action을 기반으로 state가 어떻게 변경될 지 정의
function reducer(state: State, action : Action) : State{
  switch(action.type){
    case 'up':
      return { count: state.count + 1 }
    case 'down':
      return { count: state.count - 1 > 0 ? state.count - 1 : 0 }
    case 'reset':
      return init(action.payload || { count: 0 })
    default:
      throw new Error(`Unexpected action type ${action.type}`)  
  }
}

export default function App(){
  const [state, dispatcher] = useReducer(reducer, initialState, init)

  function handleUpButtonClick(){
    dispatcher({ type: 'up' })
  }

  function handleDownButtonClick(){
    dispatcher({ type: 'down' })
  }

  function handleResetButtonClick(){
    dispatcher({ type: 'reset', payload: { count: 1}})
  }

  return(
    <div className="App">
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
      <button onClick={handleDownButtonClick}>-</button>
      <button onClick={handleResetButtonClick}>reset</button>
    </div>
  )
}
```
### useImperativeHandle
#### forwardRef
- ref를 하위 컴포넌트로 전달하고 싶을 때, 일관성을 제공하기 위해 사용
- 완전한 네이밍의 자유가 주어진 props 보다는 forwardRef를 사용하면 좀 더 확실히 ref를 전달할 것을 예측 가능
#### useImperativeHandle
부모에게서 넘겨받은 ref를 원하는대로 수정할 수 있는 훅

```javascript
const Input = forwareRef((props, ref) => {
  // ref의 동작을 추가로 정의할 수 있다.
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value)
    }),
    [props.value]
  )

  return <input ref={ref} {...props}/>
})

function App(){
  const inputRef = useRef()
  const [text, setText] = useState('')
  function handleChange(e){
    setText(e.target.value)
  }

  return (
    <>
      <Input ref={inputRef} value={text} onChange={handleChange}/>
      <button onClick={handleClick}>Focus</button>
    </>
  )
}
```

### useLayoutEffect
이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.

useLayoutEffect - 브라우저에 변경 사항 반영 되기 전 실행 
useEffect - 브라우저에 변경 사항 반영된 이후에 실행

동기적으로 발생: 리액트의 useLayoutEffect 실행이 종료될 때까지 기다린 다음 화면을 그린다는 의미

DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때 사용(ex: DOM 요소 기반으로 한 애니메이션, 스크롤 위치 제어 등)

### useDebugValue
사용자 정의 훅 내부의 내용에 대한 정보를 남길 수 있는 훅.

훅 내부에서만 실행 가능

### 훅의 규칙
1. 최상위에서만 훅을 호출해야 한다. 반복문, 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야만 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.
2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅 두 가지 경우 뿐이다. 일반 JS 함수에서는 훅을 사용할 수 없다.

훅에 대한 정보 저장은 리액트 어딘가에 있는 index와 같은 키를 기반으로 구현돼 있다(실제로는 객체 기반 링크드 리스트에 더 가깝다)

즉 훅은 항상 실행 순서를 보장받아야한다.

## 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 사용자 정의 훅
- 복잡하고 반복되는 로직을 사용자 정의 훅으로 간단하게 만들어 관리 가능
- 리액트 훅의 규칙을 따르고 react-hooks/rules-of-hooks 의 도움을 받기 위해서는 use로 시작하는 이름을 가져야 한다.

#### 언제 사용?
- 리액트에서 제공하는 훅으로만 공통 로직을 격리할 수 있을 때 사용하는 것이 좋다.
- 그 자체로는 렌더링에 영향을 미치지 못함 -> 컴포넌트 내부에 미치는 영향을 최소화할 수 있다.
- 단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶을 떄 사용한다.

### 고차 컴포넌트
- 컴포넌트 자체의 로직을 재사용하기 위한 방법
- 대표적으로는 React.memo가 있다.
  - 부모 컴포넌트가 렌더링될 때 자식 컴포넌트의 props가 변경되지 않아도 자식 컴포넌트도 함께 렌더링된다.
  - 이렇게 props가 변경되지 않아도 렌더되는 것을 방지하기 위해 만들어졌다.
  - useMemo로도 구현이 가능하나, 목적과 용도가 뚜렷한 memo를 사용하는 편이 좋다.

#### 언제 사용?
- 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용한다.
- 공통화된 렌더링 로직을 처리하기에 좋다.
- 복잡성에 주의하여 신중하게 사용해야 한다.


