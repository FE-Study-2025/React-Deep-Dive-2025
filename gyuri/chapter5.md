# 상태 관리는 왜 필요한가?

- 웹 어플리케이션에서 상태로 분류될 수 있는 것들
  1. UI: 다크/라이트 모드, 각종 input, 알림창의 노출 여부 등
  2. URL: 브라우저에서 관리되고 있는 상태값
  3. 폼(form): 로딩 중, 현재 제출되었는지, 접근 불가능한지, 값이 유효한지 등
  4. 서버에서 가져온 값: API요청

웹 애플리케이션에서 전체적으로 관리해야 할 상태를 어디에 저장하고 상태의 변화를 어떻게 감지할 것인가?

### 5.1.1 리액트 상태 관리의 역사

##### 🔷 FLUX 패턴의 등장

- 기존 MVC 패턴은 모델과 뷰가 많아질 수록 복잡도가 증가함
- 양방향이 아닌 단방향으로 데이터 흐름을 변경 하는 패턴 -> FLUX 패턴
- 용어
  - action: 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터. 액션 타입과 데이터를 정의해 디스패처로 보냄
  - dispatcher: 액션을 스토어에 보내는 역할. 콜백 함수 형태로 앞서 액션이 정의한 타입과 데이터를 스토어에 보냄
  - store: 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있음. 액션의 타입에 따라 어떻게 이를 변경할 지 정의돼 있음
  - view: 리액트의 컴포넌트에 해당하는 부분. 스토어에서 만들어진 데이터를 가져와 화면 렌더링. 상태를 업데이트 하고자 할 때 뷰에서 액션 호출
- FLUX 패턴은 사용자의 입력에 따라 데이터를 갱신하고 화면을 어떻게 업데이트해야 하는지도 코드로 작성해야 함 -> 코드의 양이 많아짐
- 데이터의 흐름은 모두 액션이라는 단방향으로 줄어드므로 데이터의 흐름을 추적하기 쉽고 코드를 이해하기 수월함

##### 🔷 리덕스의 등장

- Flux 구조를 구현하기 위해 만들어진 라이브러리 중 하나
- Elm 아키텍처를 도입

  -> Elm은 웹페이지를 선언적으로 작성하기 위한 언어임
  -> Elm아키텍처를 Flux와 마찬가지로 데이터 흐름을 3가지로 분류해 이를 단방향으로 강제해 웹 애플리케이션의 상태를 안정적으로 관리하고자 노력함

- 하나의 글로벌 상태 객체를 통해 이 상태를 하위 컴포넌트에 전파할 수 있음
- 하고자 하는 일에 비해 보일러플레이트가 많음

##### 🔷 Context API와 useContext

- Context API: 전역 상태를 하위 컴포넌트에 주입할 수 있는 API
- 상태 관리가 아닌 상태 주입을 도와주는 기능
- 렌더링을 막아주는 기능이 없으므로 사용 시 주의 필요

##### 🔷 훅의 탄생, 그리고 React Query와 SWR

- http 요청에 특화된 상태 관리 라이브러리

##### 🔷 Recoil, Zustand, Jotai, Valtio에 이르기까지

- 훅을 활용해 상태를 가져오거나 관리할 수 있는 다양한 라이브러리가 등장
- 함수형 컴포넌트에서 손쉽게 사용할 수 있음
- redux나 Mobx도 react-redux나 mobx-react-lite 등을 설치하면 동일하게 훅으로 상태를 가져올 수 있음

# 5.2 리액트 훅으로 시작하는 상태관리

### 5.2.1 가장 기본적인 방법: useState와 useReducer

- useState와 useReducer는 모두 지역 상태 관리를 위해 만들어졌음
- 컴포넌트 별로 초기화 되므로 컴포넌트에 따라 서로 다른 상태를 가지게됨
- 여러 컴포넌트에서 상태를 공유하려면 컴포넌트 트리를 재설계하는 등의 수고로움이 따름

### 5.2.2 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기

- 별도의 파일을 만들어 상태를 선언하면, 변수를 공유할 수는 있지만, 새로운 상태에 따라 리렌더링을 일으킬 수 없음
- 컴포넌트에서 리렌더링이 일어나려면 다음 작업 중 하나가 일어나야 함

  1. useState, useReducer 의 반환값 중 두 번째 인수 호출
  2. 부모 컴포넌트의 리렌더링 or 해당 컴포넌트 재실행 -> 비효율적

- 함수 외부에서 상태를 참조하고 이를 통해 렌더링까지 자연스럽게 일어나려면 다음과 같은 조건을 만족해야 함
  1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 같이 사용할 수 있어야 함
  2. 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 하고 상태가 변화될 때마다 리렌더링이 일어나 컴포넌트를 최신 상태값 기준으로 렌더링해야 함
  3. 상태가 원시값이 아닌 객체인 경우에 그 객체에 내가 감지하지 않는 값이 변하면 리렌더링이 발생해서는 안됨

```ts
export const createStore = <State extends unknown>(
  initialState: Initializer<State>
): Store<State> => {
  let state =
    typeof initialState !== "function" ? initialState : initialState();

  const callbacks = new Set<() => void>();
  const get = () => state;
  const set = (nextState: State | ((prev: State) => State)) => {
    state =
      typeof nextState === "function"
        ? (nextState as (prev: State) => State)(state)
        : nextState;

    callbacks.forEach((callback) => callback());
    return state;
  };

  const subscribe = (callback: () => void) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback);
    };
  };

  return { get, set, subscribe };
};
```

```tsx
export useStoreSelector = <State extends unknown, Value extends unknown>(store:Store<State>,selector:(state:State)=>Value)=>{
  const [state,setState] = useState(()=>selector(store.get()));

  useEffect(()=>{
    const unsubscribe = store.subscribe(()=>{
      const value = selector(store.get());
      setState(value);
    });

    return unsubscribe;
  },[store,selector]);

  return state;
}
```

- store의 subscribe로 setState를 수행하는 함수를 등록해 두었다가 createStore 내부에서 값이 변경될 때마다 subscribe에 등록된 함수를 실행
- selector(store.get())을 사용해 컴포넌트에서 필요한 값만 select해 사용할 수 있고 객체에서 변경된 값에 대해서만 수행할 수 있음

### 5.2.3 useState와 useContext를 동시에 사용해 보기

- 5.2.2의 방법은 스토어를 여러 개 생성하려면 useStore같은 훅도 동일한 개수로 생성해야 함
- context를 활용해 해당 스토어를 하위 컴포넌트에 주입하면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있게 됨
