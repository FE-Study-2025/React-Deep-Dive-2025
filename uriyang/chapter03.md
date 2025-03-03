# 3장 - 리액트 훅 깊게 살펴보기
- hook: 함수 컴포넌트가 클래스 컴포넌트의 다양한 기능을 사용하기 위해 추가됨

 ## 1 리액트의 모든 훅 파헤치기
 - 훅의 등장으로 함수 컴포넌트로 작성하면서 클래스 컴포넌보다 코드가 간결해졌다.

### 1.1 useState
- 내부의 상태를 정의하고 관리할 수 있게 해주는 훅
- 아무런 값을 넘기지 않았을 때의 초깃값은 undefined
- useReducer를 이용해 구현돼있다.
- 함수 컴포넌트 환경에서 state의 값을 유지하기 위해서는 클로저를 활용

> #### 게으른 초기화
- 게으른 초기화(lazh initialization): useState의 기본값을 변수 대신 함수로 넘기는 것
- 초깃값이 복잡하거나 무거운 연산을 연산을 포함하고 있을 때 사용 (초기화할때만 실행)
- state의 초기화를 위한 목적의 useEffect는 지양해야겠다.


### 1.2 useEffect
- 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매케니즘 (함수형 프로그래밍 참고)
- 의존성 배열의 변경된 값은 이전값과 얕은 비교를 수행한다.
- 콜백 인수로 비동기 함수가 사용 가능하다면, 결과값이 이상하게 나타날 수 있다. (race condition)

> #### 클린업 함수의 목적
- 클린업 함수는 이전 state를 참조해 실행된다.
- useEffect는 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다.
- 특정 이벤트 핸들러가 무한히 추가되는 것을 방지

> #### 의존성 배열
- 빈 배열을 둔다면 최초 렌더링 직후에 실행된 다음부터 더 이상 실행되지 않는다.
- 의존성 배열이 없는 useEffect와 일반 코드와의 차이점
  ```javascript
  // 1
  function Component() {
    console.log('렌더링됨')
  }

  // 2
  function Component() {
    useEffect(() => {
      console.log('렌더링됨')
    })
  }
  ```
  1. useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다. window 객체의 접근에 의존하는 코드를 사용해도 된다.
  2. 컴포넌트의 렌더링이 완료된 이후에 실행된다.

> #### useEffect를 사용할 때 주의할 점
- eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라
- useEffect의 첫 번째 인수에 함수명을 부여하라 (useEffect의 목적 파악)
  ```javascript
  useEffect(() => {
    logging(user.id)
  }, [user.id])

  useEffect(
    function logActiveUser() {
      logging(user.id)
    },
    [user.id]
  )
  ```
- 거대한 useEffect를 만들지 마라
  - 부수효과의 크기가 커질수록 애플리케이션 성능에 악영향을 미친다
- 불필요한 외부 함수를 만들지 마라


### 1.3 useMemo
- 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 저장된 값을 반환하는 훅
  

### 1.4 useCallback
- 인수로 넘겨받은 콜백 자체를 기억

### 1.5 useRef
- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경 가능
- useRef는 값이 변경되도 렌더링을 발생시키지 않는다
- 최조 기본값은 useRef()로 넘겨받은 인수
- useRef를 사용한 DOM 접근 예제
  ```javascript
  function RefComponet() {
    const inputRef = useRef()

    // 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undefined를 봔환한다.
    console.log(inputRef.current) // undefined

    useEffect(() => {
      console.log(inputRef.current) // <input type="text"></input>
    }, [inputRef])

    return <input ref={inputRef} type="text" />
  }
  ```


### 1.6 useContext

> #### Context란?
- 여러 depth의 복잡한 props drilling 문제를 극복하기 위해 등장한 개념

> #### Context를 함수 컴포넌트에서 사용할 수 있게 해주는 useContext 훅
- 여러 개의 provider가 있다면 가장 가까운 provider의 값을 가져온다.
```javascript
const Context = createContext<{ hello: string} | undefined>(undefined)

function ParentComponent() {
  return (
    <>
      <Context.Provider value={{ hello: 'react' }}>
        <Context.Provider value={{ hello: 'javascript' }}>
          <ChildComponent />
        </Context.Provider>
      </Context.Provider>
    </>
  )
}

function ChildComponent() {
  const value = useContext(Context)

  // react가 아닌 javascript가 반환된다.
  return <>value ? value.hello : ''</>
}
```
- useContext 내부에서 해당 콘텐스트 존재하는 환경인지 체크하는 것이 좋다
- 별도 함수로 감싸서 사용하는 것이 좋다.

> #### useContext를 사용할 때 주의할 점
- 함수 컴포넌트 내부에서 사용하면, 컴포넌트 재활용이 어려워짐 (Provider에 의존성)
- 컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 한다.
- 상태 관리를 위한 api가 아니다. (상태 주입)
  1. 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
  2. 상태 변화를 최적화할 수 있어야 한다.
- childComponent가 렌더링되지 않게 막으려면 React.memo를 써야 한다.


### 1.7 useReducer
- 복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있게 정의
- dispatcher: state를 업데이트하는 함수. action를 넘겨준다.
- 3개의 인수 (사용법 225p 참고)
  - reducer: 기본 action을 정의하는 함수
  - initialState: useReducer의 초깃값
  - init: 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수


### 1.8 useImperativeHandle

> #### forwardRef 살펴보기
- ref는 props로 쓸 수 없다
- ref를 전달하는 데 일관성을 제공
  ```javascript
  const ChildComponent = forwardRef((props, ref) => {
    useEffect(() => {
      // { current: undefined }
      // { current: HTMLInputElement }
      console.log(ref)
    }, [ref])

    return <div>안녕!</div>
  })
  
  function ParentComponent() {
    const inputRef = useRef()

    return (
      <>
        <input ref={inputRef} />
        <ChildComponent ref={inputRef} />
      </>
    )
  }
  ```
- ref를 받고자 하는 컴포넌트를 forwardRef로 감싸고, 두 번째 인수로 ref를 전달
  
> #### useImperativeHandler란?
- 부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅


### 1.9 useLayoutEffect
- 이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생
- 실행 순서
  - 리액트가 DOM을 업데이트
  - useLayoutEffect를 실행
  - 브라우저에 변경 사항을 반영
  - useEffect를 실행
- useLayoutEffect의 실행이 종료된 후 화면을 그림


### 1.10 useDebugValue
- 개발 과정에서 리액트 개발자 도구에서 볼 수 있게 사용


### 1.11 훅의 규칙
- 최상위에서만 훅을 호출해야 한다.
- 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅 두 가지 경우 뿐



## 2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 2.1 사용자 정의 훅
- 서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용
- 이름이 반드시 use로 시작하는 함수를 만들어야 한다.


### 2.2 고차 컴포넌트
- 컴포넌트 자체의 로직을 재사용하기 위한 방법 (고차함수의 일종)
- 사용자 정의 훅보다 더욱 큰 영향력을 컴포넌트에 미칠 수 있다.
- with로 시작하는 이름을 사용해야 한다.

> #### React.memo란?
- 컴포넌트의 렌더링을 방지하기 위해 만들어진 고차 컴포넌트
- useMemo를 사용할 경우 값을 반환하기 때문에 JSX 함수 방식이 아닌 {}을 사용


### 2.3 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

> #### 사용자 정의 훅이 필요한 경우
- 리액트에서 제공하는 훅으로만 공통 로직을 처리할 수 있다면 사용자 정의 훅
- 렌더링에 영향을 미치지 못하기 때문에 사용이 제한적

> #### 고차 컴포넌트를 사용해야 하는 경우
- 컴포넌트가 반환하는 렌더링 결과물에 영향을 미쳐야 할 경우
