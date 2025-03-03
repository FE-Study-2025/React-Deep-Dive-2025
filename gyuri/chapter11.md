# 🎟️ Next.js 13과 리액트 18

## 📍 app 디렉터리의 등장

- 13버전 이전 한계
  - 모든 페이지는 각각의 물리적으로 구별된 파일로 독립돼 있었음
  - 페이지 공통으로 무언가를 집어 넣을 수 있는 곳은 `_document`와 `_app`이 유일했지만 이마저도 다른 목적을 지닌 파일임
    - `_document`는 서버에서만 작동
    - `_app`에서밖에 할 수 없어 제한적

### 라우팅

- 라우팅을 정의하는 법: 13부터는 폴더명까지만 주소로 변환됨. 파일명은 무시!

#### File conventions

1. layout.js

   - 공통적인 내용(html, head 등)을 다루는 곳
   - 주소별 공통 UI 포함할 수 있음 => 레이아웃을 더욱 유연하게 구성 가능

2. page.js
   - props
     - params: `[...id]`와 같은 동적 라우트 파라미터를 사용할 경우 해당 파라미터에 값이 들어옴
     - searchParams: layout은 페이지 탐색 중에는 리렌더링을 수행하지 않기 때문에, search parameter에 의존적인 작업을 해야 한다면 반드시 page 내부에서 수행해야 함
3. error.js
   - 에러 바운더리는 클라이언트에서만 작동하므로 error 컴포넌트도 클라이언트 컴포넌트여야 함
   - error 컴포넌트는 같은 수준의 layout에서 에러가 발생할 경우 해당 에러 컴포넌트로 이동하지 않는 점
     - 아마 `<Layout><Error>{children}</Error></Layout>`과 같은 구조로 페이지가 렌더링되기 때문일 것
4. not-found.js
   - 404페이지
5. loading.js
6. route.js
   - 파일 내부에 REST API의 get, post와 같은 메서드명을 예약어로 선언해 두면 HTTP 요청에 맞게 해당 메서드를 호출
   - `route.ts`가 존재하는 폴더 내부에는 `page.tsx`가 존재할 수 없음
   - 파라미터
     - request: fetch의 Request를 확장한 Next.js만의 Request (cookie, headers, nextUrl 등등)
     - context: params만을 가지고 있는 객체. 별도 인터페이스를 제공하지 않으므로 원하는 형식으로 선언해 사용.

## 리액트 서버 컴포넌트

### 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계

- 서버 사이드 렌더링

  - 미리 서버에서 DOM을 만들어 옴 -> 클라에서는 이렇게 만들어진 DOM을 기준으로 하이드레이션 진행 -> 브라우저에서는 상태를 추적, 이벤트 핸들러를 DOM에 추가, 응답에 따라 렌더링 트리를 변경함

- Next.js SSR의 한계: 리액트가 클라이언트 중심으로 돌아가기 때문에,

  - 자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없음
  - 백엔드 리소스에 대한 직접적인 접근 불가능
  - 자동 코드 분할 불가능
  - 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어려움
  - 추상화에 드는 비용이 증가함

- CSR: 사용자의 인터랙션에 따라 다양한 것들을 제공할 수 있음 / 서버에 비해 느리고 데이터를 가져오는 것도 어려움
- SSR: 정적 콘텐츠를 빠르게 제공하고, 서버에 있는 데이터에 손쉽게 제공 가능 / 인터랙션에 따른 다양한 사용자 경험 제공의 어려움

### 서버 컴포넌트란?

- 하나의 언어, 하나의 프레임워크, 그리고 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법 (일부는 클라이언트에서, 일부는 서버에서 렌더링)
- CSR과 SSR 구조의 장점을 모두 취하고자 등장
- 서버 컴포넌트를 활용해 서버에서 렌더링할 수 있는 컴포넌트는 서버에서 완성해서 제공받은 다음, 클라이언트 컴포넌트는 서버 사이드 렌더링으로 초기 HTML을 빠르게 전달받을 수 있음

#### 종류

1. 서버 컴포넌트
   - 요청이 오면 그 순간 서버에서 딱 한 번 실행될 뿐이므로 상태를 가질 수 없음.
   - 렌더링 생명주기도 사용할 수 없음. 한번 렌더링되면 끝이기 때문
   - DB, 내부 서비스, 파일 시스템 등 서버에만 있는 데이터를 `async/await`로 접근할 수 있음
2. 클라이언트 컴포넌트
   - 브라우저 환경에서만 실행되므로 서버 컴포넌트를 불러오거나, 서버 전용 훅이나 유틸리티를 불러올 수 없음
   - 서버 컴포넌트 -> 클라 컴포넌트 -> 서버 컴포넌트는 가능 ⭐️
     - 클라 컴포넌트 입장에서 봤을 때, 서버 컴포넌트는 이미 서버에서 만들어진 트리를 가지고 있을 것이기 때문
3. 공용 컴포넌트
   - 서버 컴포넌트와 클라이언트 컴포넌트의 모든 제약을 받는 컴포넌트

### 서버 컴포넌트는 어떻게 작동하는가?

1. 서버가 렌더링 요청을 받음
2. 서버는 받은 요청에 따라 컴포넌트를 JSON으로 직렬화함
   - 서버 컴포넌트에서 클라이언트 컴포넌트로 props를 넘길 때 반드시 직렬화 가능한 데이터를 넘겨야 함
3. 브라우저가 리액트 컴포넌트 트리를 구성함

> 직렬화 가능한 데이터 : 원시 타입(Primitive types)과 일반 객체/배열
> 직렬화 불가능한 데이터 :

- function: () => console.log("함수"),
- date: new Date(),
- map: new Map(),
- set: new Set(),
- symbol: Symbol("심볼"),

## Next.js에서의 리액트 서버 컴포넌트

- `page.js`와 `layout.js`는 반드시 서버 컴포넌트여야 함

### 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps의 삭제

- 서버에서 데이터를 직접 불러올 수 있게 됨
- 컴포넌트가 비동기적으로 작동하는 것도 가능
- fetch API를 확장해 같은 서버 컴포넌트 트리 내에서 동일한 요청이 있다면 재요청이 발생하지 않도록 요청 중복을 방지함 (SWR이나 react-query와 유사)

> Pages Router vs App Router

1. Pages Router (pages 디렉토리)

- getServerSideProps, getStaticProps 등은 여전히 사용 가능
- 기존 프로젝트들의 하위 호환성을 위해 계속 지원됨

2. App Router (app 디렉토리)

- 이러한 데이터 페칭 메서드들을 사용할 수 없음
- 대신 새로운 fetch API와 서버 컴포넌트를 사용

### 정적 렌더링과 동적 렌더링

- 13버전 이전: `getStaticProps`를 활용해 정적 렌더링 가능
- 13버전 이후:

  - 정적인 라우팅에 대해서는 기본적으로 빌드 타임에 렌더링을 미리 해두고 캐싱.
    - `no-cache`옵션을 추가하면 미리 빌드해서 대기시켜두지 않음
  - 동적 라우팅에 대해서는 서버에 매번 요청이 올 때마다 컴포넌트를 렌더링함
    - 특정 주소에 대해서 캐싱하고 싶은 경우, `generateStaticParams`를 사용하면 됨

- fetch 옵션
  - `force-cache`: 데이터를 캐싱해 해당 데이터로만 관리함
  - `no-store`, `revalidate: 0`: 캐싱하지 않고 매번 새로운 데이터를 불러옴
  - `revalidate: 10`: 정해진 유효시간 동안에는 캐싱, 이 유효시간이 지나면 캐시를 파기함

### 캐시와 mutating, 그리고 revalidating

- `fetch`의 `{next: {revalidate?: number | false}}` 옵션 이용
  - 데이터의 유효한 시간을 정해두고 설정한 시간이 지나면 다시 데이터를 불러와서 페이지를 렌더링하는 것이 가능
- 페이지에 `revalidate`라는 변수를 선언해서 페이지 레벨로 정의하는 것도 가능함
  - `export const revalidate = 60`
- 캐시 무효화
  - `router.refresh()` 사용
    - 브라우저의 히스토리에 영향을 미치지 않고, 오로지 서버에서 루트부터 데이터를 전체적으로 가져와서 갱신하게 됨
    - 브라우저나 리액트의 state에는 영향을 미치지 않음

### 스트리밍을 활용한 점진적인 페이지 불러오기

- 스트리밍: 하나의 페이지가 다 완성될때까지 기다리는 것이 아니라 HTML을 작은 단위로 쪼개서 완성되는 대로 클라이언트로 점진적으로 보내는 것
  - TTFB, FCP 개선에 큰 도움을 줌
- 방법
  1. 경로에 `loading.tsx`배치
  - 자동으로 suspense를 배치함
  2. 좀 더 세분화된 제어를 하고 싶다면 `Suspense`를 직접 활용

## 웹팩의 대항마, 터보팩의 등장(beta)

- Rome, SWC, esbuild의 공통점
  - 기존에 자바스크립트로 만들어지고 제공되던 기능을 Rust나 Go 같은 다른 언어를 사용해 제공함으로써 자바스크립트 대비 월등히 뛰어난 성능을 보여줌
- 13버전에서는 웹팩의 후계자를 자처하고 있는 터보팩이 출시됨
  - 터보팩은 러스트 기반으로 웹팩 대비 700배, Vite 대비 최대 10배 빠르다고 함.

## 서버 액션(alpha)

(리액트19 에서는 [Server Functions](https://react.dev/reference/rsc/server-functions)로 이름이 바뀜)

- API를 굳이 생성하지 않더라도 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행할 수 있는 기능
  - 서버 액션을 사용하면, API 엔드포인트를 생성하지 않고도 컴포넌트 내에서 비동기 함수를 직접 정의할 수 있습니다.
- 서버 컴포넌트와 다르게, 특정 함수 실행 그 자체만을 서버에서 수행할 수 있다는 장점이 있음
- 서버 액션은 서버에서 정의되고 실행되지만, 클라이언트 컴포넌트에서 이를 트리거하는 것이 가능하다.
- 폼의 뮤테이션을 할 수 있게 해주는 넥스트의 아주 강력한 기능

[Server Actions and Mutations](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#passing-actions-as-props)

```tsx
"use server";

export async function create() {}
```

```tsx
"use client";

import { create } from "./actions";

export function Button() {
  return <button onClick={() => create()}>Create</button>;
}
```

서버 컴포넌트에서 클라이언트 컴포넌트로 직접적으로 함수를 props로 전달할 수는 없다.
대신 서버 액션 전달 가능하다.

- 서버 액션 사용하기

```tsx
// actions.ts
"use server";
export async function handleServerAction(data: string) {
  // 서버에서 실행될 로직
  console.log("서버에서 처리:", data);
}

// ServerComponent.tsx
import { handleServerAction } from "./actions";

const ServerComponent = () => {
  return <ClientButton action={handleServerAction} />;
};

// ClientButton.tsx
("use client");
const ClientButton = ({ action }) => {
  return <button onClick={() => action("데이터")}>서버 액션 실행</button>;
};
```

### form의 action

- `<form/>`에 action props를 추가해서 양식 데이터를 처리할 URI를 넘겨줄 수 있음

### input의 submit과 image의 formAction

### startTransition과의 연동

- `useTransition`에서 제공하는 `startTransition`에서도 서버 액션을 활용할 수 있음
- 서버 액션이 실행되는 동안 UI가 응답성을 유지할 수 있게 해줌

```tsx
"use client";

import { useTransition } from "react";
import { updateData } from "./actions";

function ExampleComponent() {
  const [isPending, startTransition] = useTransition();

  return (
    <div>
      {isPending ? <p>업데이트 중...</p> : null}
      <button
        onClick={() => {
          startTransition(async () => {
            await updateData(); // 서버 액션
          });
        }}
      >
        데이터 업데이트
      </button>
    </div>
  );
}
```

### server mutation이 없는 작업

- server mutation이 필요하다면 반드시 서버 액션을 `useTransition`과 함께 사용해야 하지만 별도의 server mutation을 실행하지 않는다면 바로 이벤트 핸들러에 넣어도 됨

```tsx
// Server Mutation이 필요한 경우 (useTransition 사용 필요)
const updateUserProfile = async () => {
  await db.users.update({
    // 데이터베이스 변경
    where: { id: userId },
    data: { name: newName },
  });
};

// Server Mutation이 없는 경우 (일반 이벤트 핸들러 사용 가능)
const fetchUserData = async () => {
  const data = await db.users.findFirst({
    // 단순 데이터 조회
    where: { id: userId },
  });
  return data;
};
```

### 서버 액션 사용 시 주의할 점

- 서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 없음
- 서버에서만 실행될 수 있는 자원은 반드시 파일 단위로 분리해야 함
  - 클라이언트 컴포넌트에서 서버 액션을 쓰고 싶을 때는 앞의 `startTransition` 예제처럼 'use server'로 서버 액션만 모여 있는 파일을 별도로 import해야 함
  - 서버 액션을 `import`하는 것뿐만 아니라, props 형태로 서버 액션을 클라이언트 컴포넌트에 넘기는 것 또한 가능함.

## Next.js 13 코드 맛보기

기본적으로 모든 컴포넌트는 정적으로 렌더링되며, 데이터 가져오기 방식에 따라 동작이 결정됨.

### getServerSideProps 와 비슷한 서버 사이드 렌더링 구현해보기

fetch로 변경

### getStaticProps와 비슷한 정적 페이지 렌더링 구현해보기

빌드해보면 정적 페이지로 구성되어있음

cache: 'force-cache' 옵션: 이전의 getStaticProps와 유사한 역할을 합니다. 이 옵션을 사용하면 빌드 시점에 데이터를 가져와서 캐싱합니다.
generateStaticParams(): 이전의 getStaticPaths와 유사한 역할을 하며, 동적 경로에 대한 정적 페이지를 생성할 때 사용합니다.

### 로딩, 스트리밍, 서스펜스

스트리밍과 서스펜스를 활용해 컴포넌트가 렌더링 중이라는 것을 나타낼 수 있다.

- 기본적으로 loading 지원
- Suspense로 세밀하게 쪼개서 보여줄 수 있음
