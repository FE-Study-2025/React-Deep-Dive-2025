# 10장 - 리액트 17과 18의 변경 사항 살펴보기
- 리액트를 사용하고 있는 사이트들은 일반적으로 16 버전 사용 중

## 1 리액트 17 버전 살펴보기
- 기존 코드의 수정을 필요로 하는 변경 사항을 최소화

### 1.1 리액트의 점진적인 업그레이드
- 16 -> 17로의 업데이트는 단행된 주 버전 업데이트다.
  (호환되지 않는 API가 있거나 작동방식이 달라짐)
- 17 버전부터는 점진적인 업그레이드가 가능해진다.
- 리액트 17 애플리케이션은 내부에서 리액트 16을 lazy하게 불러온다.
```javascript
// app.jsx
// 리액트 17로 작성된 애플리케이션
export default function App() {
  return <Suspense fallback={<Spinner />}>
    <AboutPage />
  </Suspense>
}

// about.jsx
// 리액트 16버전의 Greeting Component를 불러온다.
const Greeting = lazyLegacyRoot(() => import('../legacy/Greeting'));

function AbountPage() {
  const theme = useContext(ThemeContext);
  return (
    // 전체 코드는 리액트 17로 작성되지만
    <>
      <Clock />
      // 여기는 리액트 16 코드가 존재한다.
      <Greeting />
      // 리액트 16 코드 끝
    </>
  )
}
```
- 참고글: https://velog.io/@typo/how-airbnb-smoothly-upgrades-react

### 1.2 이벤트 위임 방식의 변경
- 리액트는 핸들러를 DOM 요소에 추가하는 것이 아니라, 이벤트 타입(click, change)당 하나의 핸들러를 루트에 부착 (DOM에는 noop 핸들러 추가)
- 이벤트 위임이 리액트 16 버전까지는 document에서 수행되고 있었는데, 17에서는 리액트 컴포넌트 루트로 변경
- 16버전에서 이벤트를 막는 코드를 추가하면 리액트의 모든 핸들러가 작동하지 않도록 막을 수 있었다.
  ```javascript
  <body>
    <!-- 리액트 컴포넌트 루트 -->
    <div id="main">
      ...
    </div>
  </body>
  <script>
  document.getElementById('main').addEventListener(
    'click',
    function (e) {
      e.stopPropagation()
    },
    false,
  )
  </script>
  ```


### 1.3 import React from 'react'가 더 이상 필요 없다: 새로운 JSX transform
- 17버전부터 import React 구문 없이 JSX 변환 가능
  ```javasciprt
  'use strict'

  var _jsxRuntime = require('react/jsx-runtime')

  var Component = (0, _jsxRuntime.jsx)('div', {
    ...
  });
  ```
- 기존 코드에서 import React를 삭제하려면 react-codemod 명령어 사용
  ```
  npx react-codemod update-react-imports
  ```


### 1.4 그 밖의 주요 변경 사항

> #### 이벤트 풀링 제거
  - 이벤트를 처리하기 위한 SyntheticEvent라는 이벤트 객체를 사용하는 풀링 개념이 사라졌다.
  
> #### useEffect 클린업 함수의 비동기 실행
  - 리액트 17버전부터는 화면이 완전히 업데이트(리렌더링)된 이후에 클린업 함수가 비동기적으로 실행

> #### 컴포넌트의 undefined 반환에 대한 일관적인 처리
  - 17 컴포넌트 undefined 에러 / 18 에러 X



# 2 리액트 18 버전 살펴보기
- 가장 큰 변경점은 동시성 지원

## 2.1 새로 추가된 훅 살펴보기

> #### useid
  - 컴포넌트별로 유니크한 값을 생성하는 훅
    ```javascript
    const id = useId()
    return <div>{id}</div>
    ```

> #### useTransition
  - UI 변경을 막지않고 상태를 업데이트 할 수 있는 훅
  - useTransition을 통해 처리하면, 렌더링이 블로킹되지 않음
  ```javascript
  const [isPending, startTransition] = useTransition()

  function selectTab(nextTab: Tab) {
    startTransition(() => {
      setTab(nextTab)
    })
  }

  return (
    <>
      {isPending ? ('로딩중'): <>...</>}
    </>
  )
  ```
  - 사용상 주의점
    - startTransition 내부는 반드시 상태를 업데이트하는 관련된 작업만 넘길 수 있다.
    - startTransition으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 읺애 실행이 지연될 수 있다.
    - startTransition으로 넘겨주는 함수는 반드시 동기 함수여야 한다.

> #### useDeferredValue
  - 리렌더링이 급하지 않은 부분은 지연할 수 있게 도와주는 훅
  - 디바운스와 비슷하지만, 고정된 지연 시간 없이 usedeferrerdValue로 지연된 렌더링을 수행
  - 지연된 렌더링 중단 가능
  - 사용자의 인터랙션 차단 X

> #### useSyncExternalStore
  - 동시성 이슈가 발생할 가능성이 있는 리액트 18에서, 테어링 현상을 막기 위한 훅

> #### useInserionEffect
  - CSS-in-js 라이브러리를 위한 훅
  - DOM이 실제로 변경되기 전에 동기적으로 실행된다.


## 2.2 react-dom/client

> #### createRoot
  - 기존의 react-dom에 있던 render 메서드를 대체할 메서드
  ```javascript
  // before
  ReactDOM.render(<App />, container)

  // after
  const root = ReactDOM.createRoot(container)
  root.render(<App />)
  ```

> #### hydrateRoot
  - 서버 사이드 렌더링 애플리케이션에서 하이드레이션을 하기 위한 새로운 메서드
  ```javascript
  // before
  ReactDOM.hydrate(<App />, container)

  // after
  const root = ReactDOM.hydrateRoot(container, <App />)
  ```


## 2.3 react-dom/server

> #### renderToPipeableStream
  - 리액트 컴포넌트를 HTML로 렌더링하는 메서드
  - 기존 renderToNodeStream의 단점 해결

> #### renderToReadableStream
  - renderToPipeableStream의 웹스트림 버전


## 2.4 자동 배치(Automatic Batching)
- 리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법
- 동기와 비동기 배치 작업에 일관성 보완

## 2.5 더욱 엄격해진 엄격 모드

> #### 리액트의 엄격 모드
  - 리액트 애플리케이션에서 발생할 수 있는 잠재적인 버그를 찾는 데 도움이 되는 컴포넌트
  - 엄격 모드에서 하는 작업
    - 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고
    - 문자열 ref 사용 금지
    - findDOMNode에 대한 경고 출력 (현재 사용 지양)
    - 구 Context API 사용 시 발생하는 경고
    - 예상치 못한 부작용(side-effects) 검사

> #### 리액트 18에서 추가된 엄격 모드
  - useEffect를 고의로 두 번 작동


## 2.6 Suspense 기능 강화
- 컴포넌트를 동적으로 가져올 수 있게 도와주는 기능
- 18 이전의 Suspense 문제
  - suspense 컴포넌트가 보이기 전에 useEffect가 실행되는 문제가 존재
  - 서버에서 사용 불가
- 위 두가지 문제 해결과, Suspense 내 스로틀링 추가


## 2.7 인터넷 익스플로러 지원 중단에 따른 추가 폴리필 필요
- 인터넷 익스플로러 환경에서 Promise, Symbol, Object.assgin 폴리필 필요


## 2.8 그 밖에 알아두면 좋은 변경사항
- 컴포넌트에서 undefined 반환 가능
- <Suspense fallbak={undefined}>로 null과 동일하게 처리
- renderToNodeStream 지원 중단