# 4장 - 서버 사이드 렌더링

## 1 서버 사이드 렌더링이란?

### 1.1 싱글 페이지 애플리케이션의 세상

> #### 싱글 페이지 애플리케이션이란?
- 렌더링과 라우팅에 필요한 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식
- 초기 페이지를 불러온 이후에 서버에서 HTML을 내려받지 않는다.
- 소스 보기로 HTML 코드를 봤을 때 <body /> 내부에 내용이 없다.<br/>
  (자바스크립트 코드로 삽입한 이후 렌더링)
  ```html
  <html lang="ko">
    <head>...</head>
    <body>
      <script src="runtime.fcf0559a8086856d7754.js" defer></script>
    </body>
  </html>
  ```

> #### 전통적인 방식의 애플리케이션과 싱글 페이지 애플리케이션의 작동 비교
- 전통적인 방식은 페이지 전환이 발생할 때마다 새롭게 페이지를 요청하고, 전환될 때 부자연스럽다.
- SPA는 필요한 일부 영역만 다시 그려 훨씬 매끄럽게 전환된다.

> #### 싱글 페이지 렌더링 방식의 유행과 JAM 스택의 등장
- 자바스크립트 프로임워크의 등장으로 JAM(Javascript, API, Markup) 스택이 등장했다.

> #### 새로운 패러다임의 웹서비스를 향한 요구
- 웹애플리케이션 리소스의 크기와 자바스크립트 파싱을 위해 CPU를 소비하는 시간이 증가했다.
- 웹 환경이 크게 개선됐음에도 실제 사용자들의 체감속도는 크게 차이가 없거나 오히려 느리다.


### 1.2 서버 사이드 렌더링이란?
- 서버 사이드 렌더링은 최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식

> #### 서버 사이드 렝더링의 장점
- 최초 페이지 진입이 비교적 빠르다
- 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다
  - 검색 엔진이 사이트에서 필요한 정보를 가져가는 과정
    1. 검색 엔진 로봇(머신)이 페이지에 진입한다.
    2. 페이지가 HTML 정보를 제공해 로봇이 이 HTML을 다운로드한다.
    3. 다운로드한 HTML 페이지 내부의 오픈 그래프(Open Graph)나 메타(meta) 태그 정보를 기반으로 페이지의 정보를 가져오고 검색 엔진에 저장한다.
  - 로봇은 자바스크립트 실행 없이 정적이 정보를 가져온다. 
- 누적 레이아웃 이동이 적다
  - 누적 레이아웃 이동이란 사용자에게 페이지를 보여준 이후에 어떤 HTML 정보가 추가되거나 삭제되어 화면이 덜컥거리는 것과 같은 부정적인 사용자 경험
- 사용자의 디바이스 성능에 비교적 자유롭다
- 보안에 좀 더 안전하다
  - 애플리케이션의 모든 활동이 브라우저에 노출

> #### 단점
- 소스코드를 작성할 때 항상 서버를 고려해야 한다
  - window 또는 sessionStorage와 같이 브라우저에만 있는 전역 객체 (window is not define)
- 적절한 서버가 구축돼 있어야 한다
  - 프로세스가 예기치 못하게 다운될 때를 대비해 PM2와 같은 프로세스 매니저의 도움도 필요
- 서비스 지연에 따른 문제
  - 최초 렌더링 작업이 끝나기까지 사용자에게 어떤 정보도 제공할 수 없다.


### 1.3 SPA와 SSR을 모두 알아야 하는 이융
- 서버 사이드 렌더링 역시 만능이 아니다
- 싱글 페이지 애플리케이션과 서버 사이드 렌더링 애플리케이션
  1. 가장 뛰어난 SPA는 MPA보다 낫다.
  2. 평균적인 SPA는 평균적인 MPA보다 느리다.
  > MPA에서 라우팅으로 인한 문제를 해결하기 위한 다양한 API
    - 페인트 홀딩(Paint Holding): 이전 페이지의 모습을 보여주는 기법
    - back forward cache(bfcache): 브라우저 앞,뒤 이동 시 캐시된 페이지는 노출
    - Shared Element Transitions: 라우팅 시 동일 요소가 있다면 부드럽게 전환
- 현대의 서버 사이드 렌더링
  - 최초 웹사이트 진입 시 SSR 방식으로 서버에서 완성된 HTML을 제공
  - 이후 라우팅에서는 SPA처럼 작동
  - [Universal Server Side Rendering](https://evan-moon.github.io/2018/09/25/universal-ssr/)



## 2 서버 사이드 렌더링을 위한 리액트 API 살펴보기
- Node.js와 같은 서버 환경에서만 실행 가능

### 2.1 rednerToString
- 인수로 넘겨받은 리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수
- 브라우저가 렌더링할 수 있는 HTML을 빠르게 제공하는 목적
```javascript
function SampleComponent() {
  return (
    <>
      <div>hello</div>
      ...
    </>
  )
}

const result = ReactDOMServer.renderToString(
  React.createElement('div', { id: 'root' }, <SampleComponent />),
)
```
> div#root에 존재하는 date-reactroot 속성은 hydrate 함수에서 루트를 식별하는 기준점이 된다.


### 2.2 renderToStaticMarkup
- renderToString과 유사하지만, 리액트에서만 사용하는 DOM 속성을 만들지 않는다.


### 2.3 renderToNodeStream
- renderToString과 결과물이 동일하지만 두 가지 참이점이 있다.
  - 브라우저에서 사용하는 것이 불가능하다.
  - 결과값의 타입이 다르다. (ReadableStream - utf-8로 인코딩된 바이트 스트림)
- chunk 단위로 분할해 데이터를 가져오기 위함

### 2.4 renderToStaticNodeStream
- renderToStaticMarkup의 stream 버전


### 2.5 hydrate
- 생성된 HTML 콘텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할
- hydrate가 수행된 HTML과 인수로 넘겨받은 HTML을 비교해 렌더링이 일어난다. (서버 클라이언트 두번 렌더링)



## 3 Next.js 톺아보기

### 3.1 Next.js 란?
- Next.js는 리액트 기반 프레임워크
- 디렉터리 기반 라우팅


### 3.2 Next.js 시작하기
- 구조
  - package.json: 프로젝트 구동에 필요한 모든 명령어 및 의존성 포함
  - next.config.js: 프로젝트의 환경 설정
    - /** @type {import('next').NextConfig} */
    - swc: 러스트 & 병렬 작업
  - pages/_app.tsx
    - 전체 페이지의 시작 페이지
      - 에러 바운더리를 사용
      - reset.css 같은 전역 css 선언
      - 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공
    - 초기에는 ssr, 이후 csr
  - pages/_document.tsx
    - HTML을 초기화
    - 무조건 서버에서 실행
  - pages/_error.tsx
    - 개발 모드에서는 접근 불가
  - pages/404.tsx
  - pages/500.tsx
  - pages/index.tsx
  - pages/api/hello.ts
    - API를 정의


> #### 서버 라우팅과 클라이언트 라우팅의 차이
- 최초 페이지 렌더링이 서버에서 수행


> #### 페이지에서 getServerSideProps를 제거하면 어떻게 될까?
- 정적 페이지 생성



### 3.3 Data Fetching
> #### getStaticPaths와 getStaticProps
- 어떠한 페이지를 정적인 페이지로 보여주고자 할 때 사용
- fallback 옵션: 미리 빌드하지 않은 페이지에 접근할 경우, fallback 컴포넌트를 보여주는 옵션
  
> #### getServerSideProps
- 서버에서 실행되는 함수
- window, document와 같이 브라우저 접근 객체에는 접근 불가
- useEffect 사용?

> #### getInitialProps
- 라우팅에서 서버와 클라이언트 모두에서 실행 가능한 메서드 - 권장 X



### 3.4 스타일 적용하기
> #### 전역 스타일
- _app.tsx에 필요한 스타일을 직접 import로 불러오면 애플리케이션 전체에 영향을 미칠 수 있다.

> #### 컴포넌트 레벨 CSS / SCSS와 SASS / Css-in-JS


### 3.5 _app.tsx 응용하기
- App.getInitialProps는 다른 페이지에 있는 getInitialProps를 실행해서 반환하는 역할
- getInitialProps가 있는 페이지가 getServerSideProps가 있는 페이지보다 빨리 실행


### 3.6 next.config.js 살펴보기
- swcMinify: swc를 이용해 코드를 압축할지
- poweredByHeader: false로 설정하는 것이 좋다.
- redirects:특정 주소를 다른 주소로 보내고 싶을 때
- reactStrictMode: 엄격 모드 설정 여부
- assetPrefix: 빌드 결과물을 다른 CDN 등에 업로드하고자 할 때