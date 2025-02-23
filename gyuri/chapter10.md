# 리액트 17과 18의 변경 사항 살펴보기

- 리액트 18 버전 업의 핵심은 동시성 렌더링
  - 렌더링 중간에 일시 중지한 다음, 나중에 여유가 될 때 다시 시작하거나， 진행 중인 렌더링 작업을 포기하고 새로 다시 시작할 수도 있다

## 10.1 리액트 17 버전 살펴보기

- 리액트 17버전은 16버전과 다르게 새롭게 추가된 기능은 X
- 호환성이 깨지는 변경 사항, 즉 기존에 사용하던 코드의 수정을 필요로 하는 변경 사항을 최소화했다

### 10.1.1 리액트의 점진적인 업그레이드

- 점진적인 업그레이드
- 0.3.0부터 16버전으로 오기까지 semantic version 전략에 맞춰 업데이트되고 있었음
  - 새로운 주 버전이 릴리스되면 이전 버전에서의 API 제공을 완전히 중단해 버리고, 전체 애플리케이션을 새롭게 업그레이드하기를 요구
- 리액트 17 버전부터는 점진적인 업그레이드가 가능
  - 전체 애플리케이션 트리는 리액트 17이 지만 일부 트리와 컴포넌트에 대해서만 리액트 18을 선택하는 점진적인 버전 업이 가능해짐

### 10.1.2 이벤트 위임 방식의 변경

- 둘을 같게 동작함

  - `<button ref={buttonRef}>그냥 버튼</button>`
    - 해당 버튼의 이벤트 리스너에 click으로 추가
  - `<button onClick={안녕하세요}>리액트 버튼</button>`
    - noop이라는 핸들러가 추가돼 있음. 문자 그대로(no operation) 아무런 일도 하지 않음

- 리액트에서는 이벤트를 어떻게 처리할까?

  - 리액트는 이벤트 핸들러를 해당 이벤트 핸들러를 추가한 각각의 DOM 요소에 부탁하는 것이 아니라, 이벤트 타입(click, change)당 하나의 핸들러를 루트에 부착함. 이를 이벤트 위임이라함

- 리액트는 최초 릴리스부터 이벤트 위임을 적극적으로 사용했음
  - 리액트는 이벤트 핸들러를 각 요소가 아닌 document에 연결해서 이벤트를 좀 더 효율적으로 관리함
    - ex. 루트가 div#\_\_next이고, 해당 엘리먼트에 이벤트가 달려 있음
    - 16 버전까지는 모두 document에서 수행되고 있었음
    - 17 버전부터는 이벤트 위임이 모두 document가 아닌 리액트 컴포넌트 최상단 트리, 즉 루트 요소로 바뀜
      - 이유
        - 점진적인 업그레이드 지원 / 다른 바닐라 자바스크립트 코드 또는 jQuery 등이 혼재돼 있는 경우 혼란을 방지하기 위해서임

<img width="518" alt="image" src="https://github.com/JAVACAFE-STUDY/2024-modern-react-deep-dive-study/assets/97719273/9cb4d8ca-bf6c-4166-9a09-f34dd3da9135">

### 10.1.3 import React from 'react'가 더 이상 필요 없다: 새로운 JSX transform

- 17부터는 바벨과 협력해 이러한 import 구문 없이도 JSX를 변환할 수 있게 됨

#### 구 버전 vs 17 버전

```jsx
const Component = (
  <div>
    <span>hello world</span>
  </div>
);
```

- 구 버전
  ```jsx
  var Component = React.createElement(
    "div",
    null,
    React.createElement("span", null, "hello world")
  );
  ```
- 17 버전

  ```jsx
  "use strict";

  var _jsxRuntime = require("react/jsx-runtime"); // jsx를 변환할 때 필요한 모듈인 react/jsx-runtime을 불러오는 require 구문도 같이 추가되므로 import문으로 따로 작성안해도 됨

  var Component = (0, _jsxRuntime.jsx)("div", {
    children: (0, _jsxRuntime.jsx)("span", {
      children: "hello world",
    }),
  });
  ```

### 10.1.4 그 밖의 주요 변경 사항

- 이벤트 풀링 제거
- useEffect 클린업 함수의 비동기 실행
- 컴포넌트의 undefined 반환에 대한 일관적인 처리

## 10.2 리액트 18 버전 살펴보기

### 10.2.1 새로 추가된 훅 살펴보기

- useId
  - 컴포넌트별로 유니크한 값을 생성하는 새로운 훅
  - 같은 컴포넌트여도 서로 인스턴스가 다르면 다른 랜덤 값을 만들어 냄
  - ex. 접근성 챙기기 좋음.
  - https://react.dev/reference/react/useId
- useTransition
  - UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 리액트 훅
  - 상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미룰 수 있으며, 사용자에게 조금 더 나은 사용자 경험을 제공할 수 있다.
  - 과거 리액트의 모든 렌더링은 동기적으로 작동해 느린 렌더링 작업이 있을 경우 애플리케이션 전체 적으로 영향을 끼쳤지만 useTransition과 같은 동시성을 지원하는 기능을 사용하면 느린 렌더링 과정에서 로딩 화면을 보여주거나 혹은 지금 진행 중인 렌더링을 버리고 새로운 상태값으로 다시 렌더링하는 등의 작 업을 할 수 있게 된다.
  - https://react.dev/reference/react/useTransition
- useDeferredValue
  - 리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 흑
  - 먼저 디바운스는 고정된 지연 시간을 필요로 하지만 useDeferredValue는 고정된 지연 시간 없이 첫 번째 렌 더링이 완료된 이후에 이 useDeferredValue로 지연된 렌더링을 수행
  - 만약 낮은 우선순위로 처리해야 할 작업에 대해 직접적으로 상태를 업데이트할 수 있는 코드에 접근할 수 있다면 useTransition을 사용하는 것이 좋다. 그러나 컴포넌트의 props와 같이 상태 업데이트에 관여할 수는 없고 오로지 값만 받아야 하는 상황이라면 useDeferredValue를 사용하는 것이 타당
- useSyncExternalStore
  - 외부 데이터 소스에 리액트에서 추구하는 동시성 처리가 추가돼 있지 않다면 테어링 현상이 발생할 수 있다. 그리고 이 문제를 해결하기 위한 흑이 바로 useSyncExternalStore
  - 외부에 상태 가 있는 데이터에는 반드시 useSyncExternalStore를 사용해 값을 가져외야 startTransition 등으로 인한 테어링 현상이 발생하지 않음을 알 수 있음
  - 예시
    - innerWidth 는 리액트 외부에 있는 데이터 값이므로 이 값의 변경 여부를 확인해 리렌더링까지 이어지게 하려면 useSyncExternalStore를 사용하는 것
- useInsertionEffect
  - uselnsertionEffect는 CSS-in-js 라 이브러리를 위한 흑
  - DOM이 실제로 변경되기 전에 동기적으로 실행
  - 이 흑 내부에 스 타일을 삽입하는 코드를 집어넣음으로써 브라우저가 레이아웃을 계산하기 전에 실행될 수 있게끔 해서 좀 더 자연스러운 스타일 삽입이 가능해진다.
  - uselnsertionEffect는 실제 애플리케이 션 코드를 작성할 때는 사용될 일이 거의 없으^ 라이브러리를 작성하는 경우가 아니라면 참고만 하고 실 제 애플리케이션 코드에는 가급적 사용하지 않는 것이 좋다.

### 10.2.2 react-dom/client

- createRoot
  - 기존의 react-dom에 있던 render 메서드를 대체할 새로운 메서
- hydrateRoot
  - 서버 사이드 렌더링 애플리케이션에서 하이드레이션을 하기 위한 새로운 메서드다

### 10.2.3 react-dom/server

- renderToPipeableStream
  - 리액트 컴포넌트를 HTML로 렌더링하는 메서드
  - 리액트 18에서 제공하는〈Suspense/〉와 같은 코드 분할 내지는 지연 렌더링을 서버 사이드에서 완 전히 사용하기 위해서는 renderToPipeableStream 대신에 이 메서드를 사용해야 한다.
  - 물론 실제로 renderToPipeableStream을 가지고 서버 사이드 렌더링을 만드는 경우는 거의 없긴함
- renderToReadableStream
  - renderToPipeableStream이 Node.js 환경에서의 렌더링을 위해 사용된다면，renderToReadableStream 은 웹 스트림(web stream)을 기반으로 작동한다

### 10.2.4 자동 배치

- 리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법을 의미
- 17버전 이전은 동기와 비동기 배치 작업에 일관성이 없었고，이를 보완하기 위해 리액트 18 버전부터는 루트 컴포넌트를 createRoot를 사용해
  서 만들면 모든 업데이트가 배치 작업으로 최적화할 수 있게 됐다.
- 자동 배치를 리액트 18에서도 하고 싶지 않거나 이러한 작동 방식이 기존 코드에 영향을 미칠 것으로 예상된다면 flushSync를 사용하면 된다.
- 예시
  - 버튼 클릭 한 번에 두 개 이상의 state를 동시에 업데이트한다 고 가정해 보자. 자동 배치에서는 이를 하나의 리렌더링으로 묶어서 수행할 수 있다

### 10.2.5 더욱 엄격해진 엄격 모드

- 리액트의 엄격 모드
  - 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고
  - 문자열 ref 사용 금지
  - findDOMNode에 대한 경고 출력
  - 구 Context API 사용 시 발생하는 경고
  - 예상치 못한 부작용(side-effects) 검사
- 리액트 18에서 추가된 엄격 모드
  - 향후 리액트에서는 컴포넌트가 마운트 해제된 상태에서도 (컴포넌트가 렌더링 트리에 존재하지 않는 상태에 서도) 컴포넌트 내부의 상태값을 유지할 수 있는 기능을 제공할 예정이라고 리액트 팀에서 밝혔다.
  - 컴포넌트가 최초에 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두 번째 마운트에서 이전 상태를 복원. 개발 모드에서만 적용
  - useEffect를 사용할 때 반드시 적절한 cleanup 함수를 배치해 서 반복 실행될 수 있는 useEffect로부터 최대한 자유로운 컴포넌트를 만드는 것이 좋다.

### 10.2.6 Suspense 기능 강화

- 컴포넌트가 실제로 화면 에 노출될 때 effect가 실행된다.
- Suspense에 의해 노 출이 된다면 useLay아itEffect의 effect(componentDidMount)7h 가려진다면 useLayoutEffect의 cleanup (componentWilAUnmount)이 정상적으로 실행된다.
- 서버에서도 실행할 수 있게 된다
- 스로틀링이 추가
