# 11장 - Next.js 13과 리액트 18
- Next.js의 역사를 통틀어 가장 큰 변화가 있는 릴리스

## 1 app 디렉터리의 등장
- 이전의 Next.js 12 버전까지는 공통 레이아웃을 유지할 수 있는 방법은 _app이 유일했다.
- 이 방식은 제한적이고, 각 페이지별로 서로 다른 레이아웃을 유지할 수 있는 여지도 부족

### 1.1 라우팅
> #### 라우팅을 정의하는 법
  - 폴더명까지만 주소로 변환된다. (/app/a/b -> /a/b)

> #### layout.js
  - 루트에 만든 layout은 모든 페이지에 영향을 미치는 공통 레이아웃
  - _document.jsx에서만 처리할 수 있었던 부자연스러움이 사라졌다는 장점
  - 주의점
    - layout은 app 디렉터리 내부에서 예약어다.
    - layout은  children을 props로 받아서 렌더링해야 한다.
    - layout 내부에는 반드시 export default로 내보내는 컴포넌트가 있어야 한다.
    - layout 내부에서도 API 요청과 같은 비동기 작업을 수행할 수 있다.
  ```javasciprt
  // app/layout.tsx
  import { ReactNode } from 'react'

  export default function AppLayout({ children }: { children: ReactNode }) {
    return (
      <html lang="ko">
        <head>
          <title>안녕하세요@</title>
        </head>
        <body>
          <h1>웹페이지에 오신 것을 환영합니다.</h1>
          <main>{children}</main>
        </body>
      </html>
    )
  }

  // app/blog/layout.tsx
  import { ReactNode } from 'react'

  export default function BlogLayout({ children}: { children: ReactNode }) {
    return <section>{children}</section>
  }
  ```

> #### page.js
  - app 디렉터리 내부의 예약어다.
  - export default로 내보내는 컴포넌트가 있어야 한다.
  
> #### error.js
  - 공통 에러 컴포넌트
  - error: Error 객체와 에러 바운더리를 초기화할 reset: () => vpod 함수를 props로 받는다.
  - 클라이언트 컴포넌트여야 한다.

> #### not-found.js
  - 404 페이지를 렌더링

> #### loading.js
  - Suspense를 기반으로 해당 컴포넌트가 불러오는 중임을 나타낼 때 사용

> #### route.js
  - api의 파일명 라우팅이 없어지면서, route.js(ts)로 통일
  - 이 route.ts 파일 내부에 메서드명을 선언해두면 HTTP 요청에 맞게 해당 메서드를 호출
  - route의 함수들이 받을 수 있는 파라미터는 request, context
  ```javascript
  // apps/internal-api/hello/route.ts
  import { NextRequest } from 'next/server'

  export async function GET(request: NextRequest) {
    return new Response(JSON.stringify({ name: 'hello' }), {
      status: 200,
      headers: {
        'content-type': 'application/json',
      },
    })
  }
  ```


## 2 리액트 서버 컴포넌트

### 2.1 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계
- 자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없다.
- 백엔드 리소스에 대한 직접적인 접근이 불가능
- code split이 불가능 (리액트에서는 lazy 사용)
- 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어렵다.
- 추상화에 드는 비용이 증가한다. (템플릿 언어가 아니다)


### 2.2 서버 컴포넌트란?
- 하나의 언어, 하나의 프레임워크, 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에게 컴포넌트를 렌더링할 수 있는 기법
- 클라이언트 컴포넌트는 서버 컴포넌트를 import 할 수 없다.
- 서버 컴포넌트
  - 상태를 가질 수 없다.
  - 렌더링 생명주기, 훅도 사용할 수 없다.
  - window,document 등에 접근할 수 없다.
  - 서버에만 있는 데이터를 async/await으로 접근할 수 있다.
  - 다른 서버 컴포넌트를 렌더링 하거나, html 요소를 렌더링, 클라이언트 컴포넌트를 렌더링할 수 있다.
- 클라이언트 컴포넌트
  - 서버 컴포넌트를 불러오거나, 서버 전용 훅이나 유틸리티를 불러올 수 없다.
  - 자식으로 서버 컴포넌트를 갖는 구조는 가능
  - 'use client'로 명시한다.
  - 그 외 일반적인 리액트 컴포넌트와 같다.
- 공용 컴포넌트 (shared components)
  - 서버와 클라이언트 모두에서 사용할 수 있다.


### 2.3 서버 사이드 렌더링과 서버 컴포넌트의 차이
- 서버 사이드 렌더링의 목적은 초기에 정적인 HTML을 빠르게 내려주는 데 초첨
- 서버에서 완성해서 제공받은 다음 클라이언트 컴포넌트로 초기 HTML 빠르게 전달


### 2.4 서버 컴포넌트는 어떻게 작동하는가?
1. 서버가 렌더링 요청을 받는다.
2. 받는 요청에 따라 컴포넌트를 JSON으로 직렬화(serialize)한다.
3. 브라우저가 리액트 컴포넌트 트리를 구성한다.

- 서버 컴포넌트 작동 방식의 특별한 점
  - 서버에서 클라이언트로 정보를 보낼 때 스트리밍 형태로 보냄으로써 빨리 사용자에게 결과를 보여줄 수 있다는 장점
  - 컴포넌트별로 번들링이 가능
  - 결과물이 JSON형태로 보내진 것


## 3 Next.js에서의 리액트 서버 컴포넌트

### 3.1 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps의 삭제
- 모든 요청은 fetch를 기반으로 이뤄진다.
  ```javascript
  async function getData() {
    const result = await fetch('https://api.example.com/')

    if (!result.ok) {
      throw new Error('데이터 불러오기 실패')
    }

    return result.json()
  }

  // async 서버 컴포넌트 페이지
  export default async function Page() {
    const data = await getData()

    return (
      <main>
        <Children data={data} />
      </main>
    )
  }
  ```


### 3.2 정적 렌더링과 동적 렌더링
- 정적인 라우팅에 대해서 빌드 타임에 렌더링을 미리 해두고 캐싱해 제사용할 수 있게끔 해뒀다.
- 동적 라우팅은 요청이 올 때마다 컴포넌트를 렌더링
- fetch 옵션
  - fetch(URL, { cache: 'force-cache' }): getstaticProps와 유사하게 불러온 데이터를 캐싱
  - fetch(URL, { cache: 'no-store' }).fetch(Url, { next: revalidate: 0 }): getServerSideProps와 유사하게 매범 새로운 데이터를 불러온다.
  - fetch(URL, { revalidate: 10}): getStaticProps에 revalidate를 추가한 것과 동일


### 3.3 캐시와 mutating, 그리고 revalidating
- router.refresh() 해서 갱신


### 3.4 스트리밍을 활용한 점진적인 페이지 불러오기
- HTML을 작은 단위로 쪼개서 클라이언트로 점진적으로 보내는 스트리밍이 도입


## 4 웹팩의 대항마, 터보팩의 등장(beta)
- 웹팩 대비 최대 70배, vite 대비 최대 10배 빠르다


## 5 서버 액션(alpha)
- API를 생성하지 않더라고 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행할 수 있는 기능
- 'use serve' 지시자를 선언

### 5.1 form의 action
- action props를 추가해서 이 양식 데이터를 처리할 URI를 넘겨줄 수 있다.
- 이벤트를 발생시키는 것은 클라이언트지만 함수 자체가 수행되는 것은 서버가 된다.
- server mutation
  - redirect: 특정 주소로 리다이렉 할 수 있다.
  - revalidatePath: 해당 주소의 캐시를 즉시 업데이트 한다.
  - revalidateTag: 캐시 태그는 fetch 요청 시에 다음과 같이 추가할 수 있따.
  ```javascript
  fetch('https://localhost:8080/api/something', { next: { tags: [''] }})
  ```


### 5.2 input과 submit과 image의 formAction
- input의 submit과 image의 formAction props로도 서버 액션을 추가할 수 있다.


### 5.3 startTransition과의 연동
- useTransition에서 제공하는 startTransition에서도 서버 액션을 활용할 수 있다.


### 5.4 server mutation이 없는 작업
- server mutation을 실행하지 않는다면 바로 이벤트 핸들러에 넣어도 된다.


### 5.5 서버 액션 사용 시 주의할 점
- 클라이언트 컴포넌트 내에서 정의될 수 없다.
- 서버 액션을 import하는 것뿐만 아니라, props 형태로 서버 액션을 클라이언트 컴포넌트에게 넘기는 것 또한 가능하다.



## 6 그 밖의 변화
- 라우트에서 쓸 수 있는 미들웨어 강화
- SEO를 쉽게 작성할 수 있는 기능 추가



## 7 Next.js 13 코드 맛보기

### 7.1 getServerSideProps와 비슷한 서버 사이드 렌더링 구현해 보기
- 서버 컴포넌트라면 어디든 서버 관련 코드를 추가할 수 있게 됐다.
- Next.js 13에서도 서버 사이드 렌더링과 비슷하게 서버에서 미리 페이지를 렌더링해서 내려받는 것 이 가능


### 7.2 getStaticProps와 비슷한 정적인 페이지 렌더링 구현해 보기
- getstaticProps와 getStaticPath는 사라졌지만, fetch의 cache를 이용해 구현할 수 있다.
- Incremental Satic Regeneration: 정적으로 생성된 페이지를 점진적으로 갱신하는 것

### 7.3 로딩, 스트리밍, 서스펜스
- loading이라는 에약어 지원
- suspense가 조금 더 개발자가 워하는 형태로 쪼개서 보여줄 수 있다.
