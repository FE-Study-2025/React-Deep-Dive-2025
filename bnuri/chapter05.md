# 5. 리액트와 상태 관리 라이브러리

## 5.1 상태 관리는 왜 필요한가?

상태?  애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값 (UI, URL, 폼, 서버애서 가져온 값)
(tearing: 하나의 상태에 따라 서로 다른 결과물을 사용자에게 보여주는 현상)

### 5.1.1 리액트 상태 관리의 역사
- Flux 패턴의 등장
  - 양방향 데이터 바인딩이 상태의 변화를  추적하기 어렵게 한다..
  - 상태와 그 상태의 변경에 대한 흐름과 방식을 단방향으로 하자
  - 장점: 데이터의 흐름이 액션이라는 단뱡향으로 흐르므로 추적하기 쉽고 코드 이해가 수월
  - 단점: 개발 비용 증가
  - Action -> Dispatcher -> Store -> View -> Action ...
    - Action: 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터
    - Dispatcher: 액션을 스토어로 보내는 역할. 콜백 함수 형태
    - Store: 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가짐
    - View: 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할. 뷰에서 상태 업데이트하고자 할때 액션을 호출

- 시장의 지배자 리덕스의 등장
  - Flux 구조를 구현하기 위해 만들어진 라이브러리 중 하나 + Elm 아키텍처 도입
  - Elm: 웹페이지를 선언적으로 작성하기 위한 언어 (model, view, update) Flux와 마찬가지로 데이터 흐름을 세 가지로 분류하고 이를 단방향으로 강제하여 상태 관리
  - 특징
    - 하나의 상태 객체를 스토어로 저장
    - 객체를 업데이트하는 작업을 디스패치해 업데이트 수행
    - 애플리케이션 상태에 대한 새로운 복사본을 반환한 다음, 새로운 상태 전파
    - connect로 스토어에 바로 접근할 수 있음 -> props drilling 문제 해결
    - 하고자 하는 일에 비에 보일러플레이트가 너무 많다는 단점이 있었으나 지금은 많이 간소화 됨

- Context API와 useContext
  - 리액트 16.3에서 전역 상태를 하위 컴포넌트에 주입할 수 있는 Context API 출시
  - props로 넘겨주지 않더라도 Context API를 사용하여 원하는 곳에 Context Provider로 주입 가능
  - 주의: Context API는 상태 관리가 아닌 주입을 도와주는 기능. 렌더링을 막아주는 기능 또한 존재하지 않음

- 훅의 탄생, 그리고 React Query와 SWR
  - 리액트 16.8 버전에서 함수형 컴포넌트에 사용할 수 있는 다양한 훅 API 추가
  - state를 매우 손쉽게 재사용 가능하도록 만들 수 있음
  - HTTP 요청에 특화된 상태 관리 라이브러리 React Query, SWR

- Recoil, Zustand, Jotai, Valtio에 이르기까지
  - HTTP 요청 뿐만 아니라 범용적으로 쓸 수 있는 상태 관리 라이브러리
  - 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다는 특징
  - 기존 상태 관리 라이브러리의 아쉬운 점으로 지적받던 전역 상태 관리 패러다임에서 벗어나 개발자가 원하는 만큼의 상태를 지역적으로 관리하는 것을 가능하게 함
  - 훅을 지원함으로써 함수형 컴포넌트에서 손쉽게 사용할 수 있다는 장점

## 5.2 리액트 훅으로 시작하는 상태 관리

### 5.2.1 가장 기본적인 방법: useState와 useReducer
useState의 등장으로 손쉽게 동일한 인터페이스의 상태를 생성하고 관리할 수 있게 됨

리액트의 훅을 기반으로 만든 사용자 정의 훅은 함수형 컴포넌트라면 어디서든 손쉽게 재사용이 가능

사용자 지정 훅의 한계: 훅을 사용할 때마다 컴포넌트 별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가짐(지역상태: localState)

지역 상태라는 한계 때문에 여러 컴포넌트에 걸쳐 공유하기 위해서는 컴포넌트 트리를 재설계하는 등의 수고로움 필요

### 5.2.2 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기
고려 할 점: 구독하는 컴포넌트 리렌더링 시켜야 함, 상태가 원시값이 아닌 객체인 경우 그 객체에 내가 감지하지 않는 값이 변한다 하더라도 리렌더링 발생해서는 안됨

```typescript
// store
type Initializer<State> = State | (() => State);

interface Store<State> {
  get: () => State;
  set: (newState: State | ((prev: State) => State)) => State;
  subscribe: (callback: () => void) => () => void;
}

// createStore 함수는 초기 상태를 받아 store 객체를 반환함
export const createStore = <State extends unknown>(
        initialState: Initializer<State>
): Store<State> => {
  // 1. state는 초기 상태로 설정됨
  let state =
          typeof initialState !== "function" ? initialState : initialState();
  // 2. 콜백 함수를 저장하는 Set을 초기화함
  const callbacks = new Set<() => void>();

  // 3. 현재 상태를 반환하는 get 함수를 정의함
  const get = () => state;

  // 4. 새로운 상태를 설정하고 모든 구독자에게 알리는 set 함수를 정의함
  const set = (newState: State | ((prev: State) => State)) => {
    // 4.1. 새로운 상태가 함수인지 확인하고, 함수면 이전 상태를 기반으로 새로운 상태를 생성함
    state =
            typeof newState === "function"
                    ? (newState as (prev: State) => State)(state)
                    : newState;
    // 4.2. 모든 구독자 콜백을 호출하여 상태 변경을 알림
    callbacks.forEach((callback) => callback());
    return state;
  };

  // 5. 콜백 함수를 구독하고, 구독 해지 함수를 반환하는 subscribe 함수를 정의함
  const subscribe = (callback: () => void) => {
    // 5.1. 콜백을 구독 목록에 추가함
    callbacks.add(callback);
    // 5.2. 구독 해지 함수를 반환함
    return () => {
      callbacks.delete(callback);
    };
  };

  // 6. store 객체를 반환함
  return {
    get,
    set,
    subscribe,
  };
};

```

```typescript
// useSubscription.js
import { useState, useEffect } from 'react';
import { Store } from './store';

// useSubscription 훅은 store를 받아 상태를 구독함
export const useSubscription = <State>(store: Store<State>): State => {
  // 1. 상태를 관리하기 위한 로컬 state 정의
  const [state, setState] = useState(store.get());

  // 2. 컴포넌트가 마운트될 때 구독하고, 언마운트될 때 구독 해지
  useEffect(() => {
    const handleChange = () => setState(store.get());
    const unsubscribe = store.subscribe(handleChange);
    // 구독 해지 함수 반환
    return unsubscribe;
  }, [store]);

  // 3. 현재 상태 반환
  return state;
};
출처: https://bori-note.tistory.com/35 [보리의 FE 개발 노트:티스토리]
```
useStoreSelector에 제공하는 두 번째 인수인 selector를 컴포넌트 밖에 선언하거나, 이것이 불가능하다면 useCallback으로 감싸두지 않는다면 컴포넌트가 리렌더링될 때마다 함수가 계속 재생성되어 store의 subscribe를 반복적으로 수행할 것이다.

facebook에서 만든 useSubscription : 외부에 있는 데이터를 가져와서 사용하고 리렌더링까지 정상적으로 수행되는 것 확인 가능


### 5.2.3 useState와 Context를 동시에 사용해 보기
여러 개의 스토어 가져보기 -> 리액트의 Context를 활용하여, 해당 컴포서트를 하위 컴포넌트에 주입한다면 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근 가능할 것

리액트 싱태계의 상태 관리 라이브러리들의 작동 방식 요약
1. useState, useReducer가 가지고 있는 한계, 컴포넌트 내부에서만 사용할 수 있는 지역 상태라른 점을 극복하기 위해 외부 어딘가에 상태를 둠 (컴포넌트 최상단 내지는 상태가 필요한 부모 혹은 격리된 자바스크립트 스코프 어딘가)
2. 외부의 상태 변경을 각자의 방식으로 감지해 컴포넌트의 렌더링 일으킴

### 5.2.4 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

Recoil, Jotai는 Context와 Provider, 그리고 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는 데 초점을 맞춤
Zustand는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리하는 라이브러리. 큰 스토어는 Context가 아니라 스토어가 가지는 클로저를 기반으로 생성, 스토어의 상태가 변경되면 이 상태를 구독하고 있는 컴포넌트에 전파해 리렌더링을 알리는 방식


Zustand
- 많은 코드를 작성하지 않아도 빠르게 스토어를 만들고 사용할 수 있음
- 타입스크립트 기반으로 작성돼 있어 별도 타입 설치 없이 자연스럽게 타입스크립트 사용 가능
- 미들웨어 지원 (ex 스토어 데이터를 영구히 보존할 수 있는 persist, 복잡한 객체를 관리하기 쉽게 도와주는 immer, 리덕스와 함께 사용할 수 있는 리덕스 미들웨어 등)

