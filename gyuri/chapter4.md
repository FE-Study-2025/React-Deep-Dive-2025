# 서버 사이드 렌더링이란?

### 싱글 페이지 애플리케이션 (Single Page Application; SPA)

렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식

- 페이지 전환 : 브라우저의 history.pushState와 history. replaceState
- 애플리케이션의 HTML 코드를 크롬의 소스 보기로 봤을 때는 <body/> 내부에 아무런 내용이 없다
- 렌더링에 필요한 <body/> 내부의 내용 을 모두 자바스크립트 코드로 삽입한 이후에 렌더링하기 때문이다
- 최초에 서버에서 최소한의 데이터를 불러온 이후부터는 이미 가지고 있는 자바스크립트 리소스와 브라우 저 API를 기반으로 모든 작동이 이뤄진다.
- 자바스크립트 리소스가 커지는 단점, 한번 로딩된 이후에는 서버를 거쳐 필요한 리소스를 받아올 일이 적어지기 때문에 사 용자에게 훌륭한 UI/UX를 제공한다는 장점

### 새로운 패러다임의 웹서비스를 향한 요구

- 평균 자바스크립트 리소스 크기를 살펴보면 다음과 같이 지속적으로 우상향
- 웹페이지에 소비하는 시간은 점점 증가함

### 서버사이드 렌더링이란?

- 웹페이지가 점점 느려지는 상황에 대한 문제를 해결하기 위해 버에서 페이지를 렌더링해 제공하는 기존 방식의 웹 개발 이 다시금 떠오르고 있다.

#### SSR

```mermaid
sequenceDiagram
    participant U as 사용자
    participant B as 브라우저
    participant S as 서버

    U->>B: 웹사이트 접속
    B->>S: 페이지 요청

    rect rgb(191, 223, 255)
        Note over S: SSR 프로세스
        S->>S: 1. 데이터 fetch
        S->>S: 2. React 컴포넌트 렌더링
        S->>S: 3. HTML 생성
    end

    S->>B: HTML 응답
    B->>U: 초기 페이지 표시

    rect rgb(200, 255, 200)
        Note over B: Hydration 프로세스
        B->>B: 1. JS 로드
        B->>B: 2. React 초기화
        B->>B: 3. 이벤트 핸들러 연결
    end

    Note over U,B: 이제 웹앱이 완전히 인터랙티브함

    U->>B: 사용자 상호작용
    B->>B: CSR로 UI 업데이트
```

#### CSR

```mermaid
sequenceDiagram
    participant U as 사용자
    participant B as 브라우저
    participant S as 서버

    U->>B: 웹사이트 접속
    B->>S: 초기 요청

    rect rgb(191, 223, 255)
        Note over S: 최소한의 응답
        S->>B: 빈 HTML, JS 번들 전송
    end

    rect rgb(200, 255, 200)
        Note over B: CSR 프로세스
        B->>B: 1. JS 번들 로드
        B->>B: 2. React 초기화
        B->>S: 3. API 데이터 요청
        S->>B: 4. 데이터 응답
        B->>B: 5. React 컴포넌트 렌더링
        B->>B: 6. DOM 업데이트
    end

    B->>U: 완성된 페이지 표시

    rect rgb(255, 220, 220)
        Note over U,B: 추가 상호작용
        U->>B: 사용자 액션
        B->>B: React 상태 업데이트
        B->>B: UI 리렌더링
        alt 데이터 필요
            B->>S: API 요청
            S->>B: 데이터 응답
            B->>B: UI 업데이트
        end
    end
```

#### 장점

1. 최초 페이지 진입이 비교적 빠르다
   1. 서버에서 HTTP 요청을 수행하는 것이 더 빠르기 때문에 화면 렌더링이 HTTP 요청에 의존적이거나 렌더링해야 할 HTML의 크기가 커진다면 상대적으로 서버 사이드 렌더링이 더 빠를 수 있다.
2. 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다
   1. 검색 엔진 로봇은 자바스크립트 코드를 실행하지 않고 HTML에서 정보를 가져온다.
   2. SPA에서는 따로 메타 정보를 포함한 작동 대부분이 자바스크립트에 의존하기 때문에 검색 엔진에 정보를 제공할 수 있도록 조치해야한다.
   3. SSR은 검색 엔진에 제공할 정보를 서버에서 가공해서 HTML 응답으로 제공할 수 있으므로 검색 엔진 최적화에 대응하기가 매우 용이하다.
3. 누적 레이아웃 이동이 적다
4. 사용자의 디바이스 성능에 비교적 자유롭다
5. 보안에 좀 더 안전하다

#### 단점

1. 소스코드를 작성할 때 항상 서버를 고려해야 한다
   1. 브라우저 전역 객체인 window 또는 sessionstorage
2. 적절한 서버가 구축돼 있어야 한다
   1. 가용량을 확보, 복구 전략, 요청 분산 등등
3. 서비스 지연에 따른 문제
   1. 만일 최초 랜더링에서 지연이 발생한다면 SPA에서 처럼 '로딩 중’과 같은 정보를 SSR에서는 제공할 수 없다.

### SPA와 SSR을 모두 알아야 하는이유

1. 서버 사이드 렌더링 역시 만능이 아니다
2. 싱글 페이지 애플리케이션 vs 서버 사이드 렌더링 애플리케이션
   1. 뛰어난 싱글 페이지 애플리케이션은 가장 뛰어난 멀티 페이지 애플리케이션보다 낫다 (ex. Gmail)
      1. code splitting 잘하고, 중요성 떨어지는 리소스는 lazy loading 적용하고
   2. 평균적인 싱글 페이지 애플리케이션은 평균적인 멀티 페이지 애플리케이션보다 느리다.
      1. 페인트 홀딩(Paint Holding): 같은 출처(origin)에서 라우팅이 일어날 경우 화면을 잠깐 하얗게 띄우는 대신 이전 페이지의 모습을 잠깐 보여주는 기법
      2. back forward cache(bfcache): 브라우저 앞으로 가기. 뒤로가기 실행 시 캐시된 페이지를 보여주는 기법
      3. Shared Element Transitions: 페이지 라우팅이 일어났을 때 두 페이지에 동일 요소가 있다면 해당 콘텍스트를 유지해 부드럽게 전환되게 하는 기법

### 현대의 서버 사이드 렌더링

이전까지 설명했떤 SSR은 LAMP 스택
현대에는 SSR+SPA 섞여서 사용한다.

> Next.js, Remix

- 최초 웹사이트 진입 시에는 서버 사이드 렌더링 방식으로 서버에서 완성된 HTML을 제공받는다.
- 이후 라우팅에서는 서버에서 내려받은 자바스크립트를 바탕으로 마치 싱글 페이지 애플리케이션처럼 작동한다.

## 서버 사이드 렌더링을 위한 리액트 API 살펴보기

- renderToString
  - 인수로 넘겨받은 리액트 컴포넌트를 렌더링해 HTML 문자 열(string)로 반환하는 함수
  - 최초의 페이지를 HTML로 먼저 렌더링하는 역할
- renderToStaticMarkup
  - renderToString과 유사하게 리액트 컴포넌트를 기준으 로 HTML 문자열을 만든다
  - 루트 요소에 추가한 data- reactroot와 같은 리액트에서만 사용하는 추가적인 DOM 속성을 만들지 않는다
- [renderToNodeStream](https://18.react.dev/reference/react-dom/server/renderToNodeStream) Deprecated!
  - React 공식 문서에 따르면: renderToPipeableStream 으로 마이그레이션하라고함
    > As of React 18, this method buffers all of its output, so it doesn't actually provide any streaming benefits. This is why it's recommended that you migrate to renderToPipeableStream instead.
  - renderToString과 renderToStaticMarkup은 브라우저에 서도 실행할 수는 있지만 renderToNodeStream은 브라우저에서 사용하는 것이 완전히 불가능하다
  - renderToNodeStream의 결과물은 Node.js의 ReadableStream
  - 큰 크기의 데이터를 청크 단위로 분리해 순차적으로 처리할 수 있다
- [renderToStaticNodeStream](https://18.react.dev/reference/react-dom/server/renderToStaticNodeStream)
  - 공식문서에 deprecated 표시는 없지만 사용하지 않는 것이 좋다. 관련 [discussion](https://github.com/reactwg/react-18/discussions/22)
  - renderToNodeStream과 결과물은 동일하지만 리액트 자바스크립트에 필요한 리액트 속성이 제공되지 않는다.
- hydrate
  - renderToString과 renderToNodeStream으로 생성된 HTML 콘텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할
  - React 18에서는 hydrateRoot 함수로 대체됨
  - Next.js와 같은 프레임워크를 사용할 경우에는 이 과정이 자동으로 처리되므로 직접 hydrateRoot를 호출할 필요가 없음.

### React 19에서 server API는 4개가 제공됨

- [renderToPipeableStream](https://react.dev/reference/react-dom/server/renderToPipeableStream)
- [renderToReadableStream](https://react.dev/reference/react-dom/server/renderToReadableStream)
- [renderToStaticMarkup](https://react.dev/reference/react-dom/server/renderToReadableStream)
- [renderToString](https://react.dev/reference/react-dom/server/renderToString)

- [Next.js 아닌 React SSR에 대하여](https://cheri.tistory.com/298)

### SSR에서의 Fallback 처리 과정

fallback UI도 SSR로 처리됨.

1. **초기 HTML 스트림**

```jsx
// 서버에서 최초로 보내는 HTML에 포함됨
<Layout>
  <div>
    <HeaderSkeleton /> {/* fallback UI */}
    <ArticleSkeleton /> {/* fallback UI */}
    <CommentsSkeleton /> {/* fallback UI */}
  </div>
</Layout>
```

2. **순차적 콘텐츠 스트리밍**

```jsx
// 데이터가 준비되면 각각의 fallback이 실제 컴포넌트로 교체됨
<Layout>
  <Header /> {/* 첫 번째로 준비된 컴포넌트 */}
  <ArticleSkeleton /> {/* 아직 로딩 중 */}
  <CommentsSkeleton /> {/* 아직 로딩 중 */}
</Layout>
```

### 중요한 점

1. **Fallback UI는 서버에서 즉시 렌더링**

   - 로딩 상태를 나타내는 스켈레톤 UI나 스피너가 초기 HTML에 포함됨
   - 사용자는 빈 화면 대신 로딩 UI를 먼저 보게 됨

2. **점진적 교체**

   - 실제 컨텐츠가 준비되면 서버가 해당 부분만 추가로 스트리밍
   - JavaScript를 통해 fallback UI를 실제 컨텐츠로 자연스럽게 교체

3. **SEO 관점**
   - 검색 엔진은 초기 HTML의 fallback UI를 볼 수 있음
   - 점진적으로 교체되는 실제 컨텐츠도 HTML 스트림에 포함되므로 SEO에 문제없음

이런 방식으로 작동하기 때문에 사용자는:

1. 빈 화면을 보는 대신 의미 있는 로딩 UI를 먼저 보고
2. 준비되는 컨텐츠를 점진적으로 볼 수 있어서
   더 나은 사용자 경험을 제공받을 수 있습니다.

### 서버 사이드 렌더링 예제 프로젝트

간단한 Todo앱

- index.tsx
  - 애플리케이션의 시작점으로，hydrate가 포함돼 있다
  - 이 파일의 목적은 서버로부터 받은 HTML을 hydrate를 통해 완성된 웹 애플리케이션으로 만드는 것
  - hydrate는 서버에서 완성한 HTML과 하이드레이션 대상이 되는 HTML의 결과물이 동일한지 비교하는 작업을 거친다.
- App.tsx
  - 리액트 애플리케이션의 시작점
- Todo.tsx
  - 투두앱 내용
- index.html

```typescript
  <!D0CTYPE html>

  <head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" /> <title>SSR Example</title>

  </head> <body>

  __placeholder__
  <script src="https://unpkg.com/react@17.0.2/umd/react.development.js"x/script>
  <script src="https://unpkg.com/react-dom17.0.2/umd/react-dom.development.js"x/script> <script src="/browser.js"x/script>

  </body> </html>
```

- server.ts
  - 서버에서는 사용자의 요청 주소에 따라 어떠한 리소스를 내려 줄지 결정하는 역할을 한다
  - 서버 사이드 렌더링을 위해 이 파일에서 리액트 트리를 만드는 역할도 담당
  - createServer
    - http 모듈을 이용해 간단한 서버를 만들 수 있는 Node.js 기본 라이브러리
    ```typescript
    function main() { createServer(serverHandler).listen(PORT, () => {.
    console.log('Server has been started ${P0RT}. })
    ```

````

	- serverHandler
		- serverHandler는 createServer로 넘겨주는 인수로，HTTP 서버가 라우트(주소)별로 어떻게 작동할지를 정의하는 함수
  - webpack.config.js
    - 웹팩 설정 파일

### Next.js
Vercel이라는 미국 스타트업에서 만든 리액트 기반 서버 사이드 렌더링 프레임워크다.
다른 프레임워크에 비해 사용자도 많고，모기업인 Vercel의 전폭적인 지원을 받고 있기도 하며，Next.js뿐만 아니라 SWR, SWC, Turbopack, Svelte 등 웹 생태계 전반에 영향력 있는 프 로젝트를 계속해서 개발하거나 인수했으며，또 꾸준히 새로운 기능을 추가해서 릴리스하고 있기 때문에 리액 트 기반 프로젝트에서 Next.js를 선택하는 것은 합리적인 선택이라 할 수 있다.

[@remix-run/react vs @shopify/hydrogen vs next](https://npmtrends.com/@remix-run/react-vs-@shopify/hydrogen-vs-next)

- next가 여전히 높긴한데... 용량이 어마어마하다

#### Next.js 시작하기

- package.json
	- next, eslint-config-next 등을 확인할 수 있다
- next.config.js

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
reactStrictMode: true,
swcMinify: true,
}

module.exports = nextConfig
````

reactStrictMode : 리액트의 엄격 모드와 관련된 옵션 [링크](https://ko.legacy.reactjs.org/docs/strict-mode.html)
swcMinify : SWC를 기반으로 코드 최소화 작업을 할 것인지 여부를 설정하는 속성. 번들링과 컴파일을 더욱 빠르게 수행하기 위해 만들어졌다. 바벨의 대안.

pages/app.tsx : 애플리케이션의 전체 페이지의 시작점, 공통으로 설정해야 하는 것들을 여기에서 실행

- 에러 바운더리를 사용해 애플리케이션 전역에서 발생하는 에러 처리
- reset.css 같은전역CSS 선언
- 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공 등
- 최초에는 서버 사이드 렌더링을，이후에는클라이언트에서 \_app.tsx의 렌더링이 실행된다

- \_document.tsx
- `<html>`이나`〈body〉`에 DOM 속성을 추가하고 싶을때 사용
- document.tsx는 무조건 서버에서 실행된다
- Next.js에서는 두 가지 head가 존재한다
  - 첫 번째는 next/document에서 제공하는 head
    - 오직 document.tsx에서만 사용할 수 있다.
  - 두 번째는 next/head에서 제공하는 head
- **CSS-in-JS의 스타일을 서버에서 모아 HTML로 제공하는 작업**

\_app.tsx vs \_document.tsx
\_app.tsx: Next.js를 초기화하는 파일, 설정과 관련된 코드를 모아두는 것, 서버와 클라이언트 모두에서 렌더링 될 수 있다.
\_document.tsx: 웹사이트의 뼈대가 되는 HTML 설정과 관련된 코드를 추가하는 곳

## 서버 라우팅과 클라이언트 라우팅의 차이

Next.js는 서버 라우팅과 클라이언트 라우팅을 모두 지원한다.

- a태그로 이동하는 경우
  - webpack, framework, main, hello 등 페이지를 만드는 데 필요한 모든 리소스를 처음부터 다 가져온다.
- next/link로 이동하는 경우 - 서버 사이드 렌더링이 아닌，클라이언트에서 필요한 자바스크립트만 불러온 뒤 라우팅하는 클라이언트 라우팅/렌더링 방식으로 작동한다 - Next.js는 서버 사이드 렌더링의 장점，즉 사 용자가 빠르게 볼 수 있는 최초 페이지를 제공한다는 점과 싱글 페이지 애플리케이션의 장점인 자연스러운 라우팅이라는 두 가지 장점을 모두 살리기 위해 이러한 방식으로 작동

  > `<a>` 대신`<Link〉`를사용한다.
  > window, location.push 대신 router.push를 사용한다.

- getServerSideProps가 없는 경우 정적 페이지로 분류됨
  - 이 경우 빌드 시점에 페이지를 미리 렌더링해서 정적 페이지로 만든다
  - window 처리를 모두 object로 바꾼 후 미리 트리쉐이킹을 하기 때문

### Data Fetching

Next.js에서는 서버 사이드 렌더링 지원을 위한 몇 가지 데이터 불러오기 전략

- pages/의 폴더에 있는 라우팅이 되는 파일에서만 사용할 수 있다
- 반드시 정해진 함수명으로 export를 사용해 함수를 파일 외부로 내보내야 한다

#### getStaticPaths & getStaticProps

- pages/의 폴더에 있는 라우팅이 되는 파일에서만 사용할 수 있다
- 정적인 페이지 제공을 위해 사용

/pages/post/[id] 라는 페이지가 있다면,

- getStaticPaths는 /pages/post/[id]가 접근 가능한 주소를 정의하는 함수
  - fallback 옵션의 경우 미리 빌드해야 할 페이지가 너무 많은 경우에 사용 가능
- getStaticProps는 앞에서 정의한 페이지를 기준으로 해당 페이지로 요청이 왔을 때 제공할 props를 반환하는 함수

#### getServerSideProps

- 서버에서 실행되는 함수이며 해당 함수가 있다면 무조건 페이지 진입 전에 이 함수를 실행한다.
- 응답값에 따라 props를 반환하거나 다른 페이지로 리다이렉트 시킬 수 있다.
- script 형태로 작성되어있다.

  - 서버에서 fetch 등으로 렌더링에 필요한 정보를 가져온다.
  - 가져온 정보를 기반으로 HTML을 완성한다.
  - 완성된 HTML을 클라이언트에 제공한다.
  - 클라이언트는 hydrate 작업을 한다. DOM에 리액트 라이프 사이클과 이벤트 핸들러를 추가하는 작업
  - hydrate로 만든 리액트 컴포넌트 트리와 서버에서 만든 HTML이 다르면 불일치 에러를 뱉는다 (suppressHydrationWarning).
  - 이 작업은 fetch 등을 이용해 정보를 가져와야 한다.

- JSON.stringfy() 로 직렬화할 수 있는 값만 제공해야함 (class나 Date 같은 객체는 직렬화 불가)
- 이 함수는 사용자가 매 페이지를 호출할 때 마다 실행되고, 실행이 끝나기 전 까지는 사용자에게 어떠한 HTML도 보여줄 수 없다. 따라서 내부에서 실행하는 내용은 최대한 간결하게 작성하기 위해 최초에 보여줘야하는 데이터가 아니라면, 클라이언트에서 호출하는 것이 유리하다.

#### getInitialProps

- getStaticProps나 getServerSideProps가 나오기 전에 사용할 수 있었던 유일한 페이지 데이터 불러오기 수단
- [공식문서](https://nextjs.org/docs/pages/api-reference/functions/get-initial-props)에서는 legacy이기 때문에 사용하지 않는 것을 권장

리액트의 서버 사이드 렌더링을 하는 작동 순서

1. 서버에서 fetch 등으로 렌더링에 필요한 정보를 가져온다.
2. 1 번에서 가져온 정보를 기반으로 HTML을 완성한다.
3. 2번의 정보를 클라이언트(브라우제에 제공한다.
4. 3번의 장보를 바탕으로 클라이언트에서 hydrate 작업을 한다. 이 작업은 D O M 에 리액트 라이프사이클과 이벤트 핸들러 를 추가하는 작업이다.
5. 4번 작업인 hydrate로 만든 리액트 컴포넌트 트리와 서버에서 만든 HTMLO| 다르다면 불일치 에러를 뱉는다 (suppressHydrationWarning).
6. 5번 작업도 번과 마찬가지로 fetch 등을 이용해정보를 가져와야 한다.

### 스타일 적용하기

- 전역 스타일
- 컴포넌트 레벨 CSS
- SCSS와 SASS
- CSS—in—JS : styled-jsx, styled- components, Emotion, Linaria

styled-components의 스타일을 Next.js에 추가하려면 다음과 같은 코드가 필요하다. \_document.tsx가 없 다면 해당 파일을 만든 후 다음과 같이 추가해 보자.

```typescript
import Document, {
Html,
Head,
Main,
NextScript, Documentcontext, DocumentlnitialProps,

} from 'next/document'
import { ServerStyleSheet } from 'styled-components'

export default function MyDocument() {
```

- 프로덕션 모드에서 빌드하면 `<style />` 내부가 비어있다. styled-components가 프로덕션 모드에선는 SPEEDY_MODE를 사용하기 떄문.
  CSSOM트리에 직접 스타일을 넣는다. 확인하고싶다면 document.styleSheets를 활용하면 된다.

### next.config.js 살펴보기

- Next.js 실행에 필요한 설정을 추가할 수 있는 파일
- basePath: URL을 위한 접두사(prefix)
- swcMinify: swc를 이용해 코드를 압축할지
- poweredByHeader: Next.js는 응답 헤더에 X-Power-by: Next.js 정보를 제공(true)
- redirects: 특정 주소를 다른 주소로 보내고 싶을 때 사용
- reactStrictMode: 리액트에서 제공하는 엄격 모드
- assetPrefix: 빌드된 결과물을 동일한 호스트가 아닌 다른 CDN 등에 업로드하고자 한다면 해당 CDN 주소를 명시
