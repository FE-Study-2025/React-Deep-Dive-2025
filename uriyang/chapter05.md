# 5장 - 리액트와 상태 관리 라이브러리

## 1 상태 관리는 왜 필요한가?
- 상태란?: 애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값

### 1.1 리액트 상태 관리의 역사

> #### Flux 패턴의 등장
  - Flux 패턴: 데이터 흐름을 단방향으로 제안 (그림 5.3 참고)
    - 액션(action): 어떤 작업을 처리할 액션과 데이터
    - 디스패처(dispatcher): 액션을 스토어에 보내는 역할
    - 스토어(store): 상태과 상태 변경 메서드
    - 뷰(view): 리액트 컴포넌트에 해당
  - 사용자의 입력에 따라 데이터를 갱신과 화면 업데이트를 정의해야 되어 코드의 양이 많아진다.

> #### 시장 지배자 리덕스의 등장
  - Flux 구조와 Elm 아키텍처 도입
    - 모델(model): 애플리케이션의 상태
    - 뷰(view): 모델을 표현하는 HTML
    - 업데이트(update): 모델을 수정하는 방식
  - 하나의 객체를 스토어에 저장해두고, 이 객체를 디스패치를 통해 업데이트 한다.
  - props depth가 깊어지는 문제 해결
  - 하나의 상태를 바꿀 때 해야 할 일이 많다.

> #### Context API와 useConext
  - 리액트 16.3에서 Context API 출시
  ```javascript
  const CounterContext = createContext<Counter | undefined>(undefined)

  class CounterComponent extends Component {
    render() {
      return (
        <CounterContext.Consumer>
          {(state) => <p>{state?.count</p>}
        </CounterContext.Consumer>
      )
    }
  }

  class DummyParent extends Component {
    render() {
      return (
        <>
          <CounterComponent />
        </>
      )
    }
  }

  export default class MyApp extends Component<{}, Counter> {
    state = { count: 0 }

    componentDidMount() {
      this.setState({ count: 1 })
    }

    handleClick = () => {
      this.setState((state) => ({ count: state.count + 1 }))
    }

    render() {
      return (
        <CounterContext.Provider value={thid.state}>
          <button onClick={this.handleClick}>+</button>
          <DummyParent />
        </CounterContext.Provider>
      )
    }
  }
  ```

> #### 훅의 탄생, 그리고 React Query와 SWR
  - 16.8 버전에서 함수 컴포넌트에 사용할 수 있는 다양한 훅 API 추가
  - SWR
  ```javascript
  import useSWR from 'swr'

  const fetcher = (url) => fetch(url).then((res) => res.json())

  export default function App() {
    const { data, error } = useSWR('https://api.github.com/repos/vercel/swr', fetcher)

    if (error) return 'An error has occurred.'
    if (!data) return 'Loading...'

    return (
      <div>
        <p>{JSON.stringify(data)}</P>
      </div>
    )
  }
  ```
  - 동일한 키로 호출했을 때, useSWR이 관리하고 있는 캐시 값 활용

> #### Recoil, Zustand, Jotai, Valtio에 이르기까지
  ```javascript
  // Recoil
  const counter = atom({ key: 'count', default: 0 })
  const todoList - useRecoilValue(counter)

  // Jotai
  const countAtom = atom(0)
  const [count, setCount] = useAtom(countAtom)

  // Zustand
  const useCounterStore = create((set) => ({
    counter: 0;
    increase: () => set((state) => ({ count: state.count + 1 })),
  }))
  const count = useCounterStore((state) => state.count)

  // valtio
  const state = proxy({ count: 0 })
  const snap = useSnapshot(state)
  state.count++
  ```



## 2 리액트 훅으로 시작하는 상태 관리

### 2.1 가장 기본적인 방법: useState와 useReducer
- useState: useReducer로 구현
```javascript
function useCounter(initCount: number = 0) {
  const [counter, setCounter] = useState(initCount)

  function inc() {
    setCounter((prev) => prev + 1)
  }

  return { counter, inc }
}
```
- useReducer
```javascript
function useStateWithUseReducer<T>(initialState: T) {
  const [state, dispatch] = useReducer((prev: T, action: Initializer<T>) => typeof acton === 'function' ? action(prev) : action, initialState,
  )

  return [state, dispatch]
}
```
- useState, useReducer의 한계는 명확

### 2.2 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기
- 함수 컴포넌트에서 리렌더링을 하려면?
  - useState, useReducer의 반환값 중 두 번째 인수가 호출된다.
  - 부모 함수(부모 컴포넌트)가 리렌더링되거나 해당 함수가 다시 실행돼야 한다.
- createStore 구현 코드
  - store 초깃값 stale 또는 게으른 초기화 함수를 받아 초기화
  - callbacks를 set으로 선언
  - get, set을 만들기
  - subscribe는 callbacks Set에 callback을 등록할 수 있는 함수
  - get, set, subscribe를 하나의 객체로 반환
- useStore 훅
  ```javascript
  export const useStore = <State extends unknown>(store: Store<State>) => {
    const [state, setState] = useState<State>(() => store.get())

    useEffect(() => {
      const unsubscribe = store.subscribe(() => {
        setState(store.get())
      })
      return unsubscribe
    }, [store])

    return [state, store.set] as const
  }
  ```

### 2.3 useState와 Context를 동시에 사용해 보기
- Context를 활용해 해당 스토어를 하위 컴포넌트에 주입한다면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있게 된다.

### 2.4 싱태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

> #### 페이스북이 만든 상태 관리 라이브러리 Recoil
  - 페이스북에서 만든 상태 관리 라이브러리
  - RecoilRoot
    - 애플리케이션의 최상단에 선언
    - Recoil에서 생성되는 상태값을 저장하기 위한 스토어를 생성
    - 값이 변경되면 하위 컴포넌트에 알린다.
  - atom
    - Recoil의 최소 상태 단위
      ```javascript
      // Atom 선언
      const statementsAtom = atom<Array<Statement>>({
        key: 'statements',
        default: InitialStatements,
      })
      ```
  - useRecoilValue
    - atom의 값을 읽어오는 훅
  - useRecoilState
    - useState와 유사하게 값을 가져오고, 값을 변경할 수도 있는 훅

> #### Recoil에서 영감을 받은, 그러나 조금 더 유연한 jotai
  - Recoil의 atom 모델에 영감을 받아 만들어진 상태 관리 라이브러리
  - 개발자들이 메모이제이션이나 최적화를 거치지 않아도 리렌더링이 발생되지 않도록 설계
  - atom
    - 최소 단위의 상태
    - 파생된 상태까지 만들 수 있다
  - useAtomValue
    - 항상 최신 값 유지
  - useAtom
    - useState와 동일한 형태의 배열을 반환
> #### 작고 빠르며 확장에도 유연한 Zustand
  - Redux에 영감을 받아 만들어졌다.
  - 간단한 사용법
    ```javascript
    import { create } from 'zustand'

    const useCounterStore = create((set) => ({
      count: 1,
      inc: () => set((state) => ({ count: state.count + 1 })),
      dec: () => set((state) => ({ count: state.count - 1 })),
    }))

    function Counter() {
      const { count, inc, dec } = useCounterStore()
      return (
        <div class="counter">
          <span>{count}</span>
          <button onClick={inc}>up</buttom>
          <button onClick={dec}>down</buttom>
        </div>
      )
    }
    ```
  - 특징
    - 많은 코드를 작성하지 않아도 빠르게 스토어를 만들고 사용할 수 있다는 장점