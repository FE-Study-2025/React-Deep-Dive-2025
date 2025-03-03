# 11. Next.js 13과 리액트 18

## 11.1 app directory의 등장

### 13 버전 이전 페이지
- _document: 페이지에서 쓰이는 html, body 태그 수정, SSR 시 styled-components와 같은 일부 CSS-in-JS를 지원하기 위한 코드 삽입하는 제한적인 용도로 사용됨
- _app: 페이지 초기화 용도
  - 페이지 변경 시 유지하고 싶은 레이아웃
  - 페이지 변경 시 상태 유지
  - componentDidCatch를 활용한 에러 핸들링
  - 페이지간 추가적인 데이터 삽입
  - global CSS 주입

### 13 버전
기존 /pages로 정의하던 라우팅 방식이 /app 디렉토리로 이동 

파일명으로 라우팅하는 것이 불가능해짐. 파일명은 다음의 예약어로 제한됨

- layout.js
    - 루트에 하나의 layout - 웹 페이지 시작에 필요한 공통 코드 삽입
    - 주소별 공통 UI
    - 기존 _app, _document와는 달리 오로지 자신과 자식 라우팅에만 영향 미침
    - children을 props로 받아 렌더링
    - 페이지 탐색 중 리렌더링 수행하지 x
- page.js
  - params, searchParams 를 props로 받음
- error.js
  - Error, reset(에러 바운더리를 초기화) props로 받음
  - 에러 바운더리는 클라이언트에서만 작동(client component)
  - 루트 에러 처리는 global-error.js로
- not-found.js
  - 서버 컴포넌트
- loading.js
  - suspense 기반으로 컴포넌트 불러오는 중임을 나타냄
- route.js
  - REST API의 get, post와 같은 메서드명을 예약어로 선언해 두면 HTTP 요청에 맞게 해당 메서드를 호출하는 방식으로 동작
  - route.ts가 존재하는 폴더 내부에는 page.tsx가 존재할 수 없다.

## 11.2  리액트 서버 컴포넌트
### 11.2.1 기존 리액트 컴포넌트와 SSR의 한계
  - 자바스크립트 번들 크기가 0인 컴포넌트 만들 수 x
  - 백엔드 리소스에 대한 직접적인 접근이 불가능
  - 자동 코드 분할 불가능
  - 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어려움
  - 추상화에 드는 비용 증가(결과물만 있으면 되는데)
### 11.2.2 서버 컴포넌트란?
  - 서버, 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법
  - 요청이 오면 그 순간 딱 한 번 실행될 뿐이므로 상태를 가질 수 x(useState, useReducer 등)
  - 렌더링 생명주기를 사용할 수 x(useEffect 등)
  - 서버에서만 실행되기 때문에 DOM API, window, document 등에 접근할 수 x
  - 데이터베이스, 내부 서비스 등 서버에만 있는 데이터를 sync/await으로 접근가능
  - 다른 서버 컴포넌트, 엘리먼트, 클라이언트 컴포넌트를 렌더링 가능
### 11.2.3 서버 사이드 렌더링과 서버 컴포넌트의 차이
  - SSR
    - 응답받은 페이지 전체를 HTML로 렌더링하는 과정을 서버에서 수행 -> 결과를 클라이언트에 전달
    - 이후 하이드레이션 과정을 거쳐 결과물 확인, 이벤트 붙이는 등의 작업 수행
    - 따라서 여전히 클라이언트에서 자바스크립트 코드를 다운로드, 파싱, 실행하는 데 비용이 듦
- 서버 컴포넌트 작동 방식
  - 서버가 렌더링 요청을 받음
    - 서버는 받은 요청에 따라 컴포넌트를 JSON으로 직렬화
      - 서버에서 렌더링 할 수 있는 것은 직렬화
      - 클라이언트 컴포넌트로 표시된 부분은 플레이스 홀더 형식으로 비워둠
    - 브라우저는 이 결과물을 역직렬화 후 렌더링 수행
    - 브라우저가 리액트 컴포넌트 트리 구성 -> 브라우저 DOM에 커밋
- 서버 컴포넌트 작동 방식의 특별한 점
  - 스트리밍 형태로 전달 -> 결과물을 보다 빠르게 보여줄 수 있음
  - 컴포넌트들이 하나의 번들러 작업에 포함돼 있지 않고 컴포넌트별로 번들링이 별개로 되어 있음
  - SSR과 다르게 HTML이 아닌 JSON 형태로 보내짐
## 11.3 Next.js에서의 리액트 서버 컴포넌트
- 새로운 fetch 도입
  - 브라우저 네이티브 fetch API 확장 -> 컴포넌트 트리 내 동일한 요청이 있다면 재요청 방지(캐싱)
- 정적 렌더링과 동적 렌더링
  - 정적 라우팅 - 기본적으로 빌드 타임에 렌더링을 미리 해두고 캐싱해 재사용
  - 동적 라우팅 - 서버에 매번 요청이 올 때마다 컴포넌트를 렌더링
  - fetch에 no-cache/revalidate 0 등의 캐싱하지 않겠다는 설정을 하면 빌드하여 대기시켜 두지 않고 요청이 올때마다 fetch 요청 이후 렌더링
  - 또한, Next.js가 제공하는 next/headers 나 next/cookie 같은 함수 사용한다면 동적 연산을 바탕으로 결과 반환하는 것으로 인식해 정적 렌더링 대상에서 제외
  - generateSataicParams를 활용하여 동적인 주소를 캐싱 가능
- 스트리밍을 활용한 점진적인 페이지 불러오기
  - 전체 페이지를 한 번에 그리는 것이 아닌 부분 단위로 준비되는 것부터 렌더링
  - loading.js 또는 Suspense 배치
- 서버 액션
- API를 생성하지 않고 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행
  - useServer
  - only async function
  - 서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 x
  - 서버 액션을 import 하는 것 뿐만 아니라 props 형태로 서버 액션을 클라이언트 컴포넌트에 넘기는 것 또한 가능

## 11.5 서버 액션
- API를 굳이 생성하지 않더라도 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행할 수 있는 기능
- 서버컴포넌트와 다르게 특정 함수 실행 그 자체만을 서버에서 수행할 수 있다.
- 함수 내부 또는 파일 상단에 “use server” 지시자를 선언해야 한다. + 함수는 반드시 async

### form의 action
다음은 서버 액션으로 form.action에 handleSubmit이라는 서버 액션을 만들어 props로 넘겨줌

이벤트 발생은 클라이언트지만 실제로 함수는 서버에서 실행됨

서버 액션을 실행하면 클라이언트에서는 현재 라우트 주소와 ACTION_ID만 보냄

서버에서는 요청받은 라우트 주소와 ACTION_ID를 바탕으로 실행해야 할 내용을 찾고 직접 실행

'use server'로 선언돼 있는 내용은 빌드 시점에 미리 ㅡㄹ라이언트에서 분리시키고 서버로 옮김. 클라인언트 번들링 결과물에 포함시키지 않음

```javascript
export default function Page() {
  async function handleSubmit() {
    "use server";

    console.log("해당 작업은 서버에서 수행하기에 CORS 이슈가 없음");

    const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
      method: "POST",
      body: JSON.stringify({
        title: "foo",
        body: "bar",
        userId: 1,
      }),
      headers: {
        "Content-Type": "application/json; charset=UTF-8",
      },
    });

    const result = await response.json();
    console.log(result);
  }

  return (
    <form action={handleSubmit}>
      <button type="submit">form 요청 보내보기</button>
    </form>
  )
}
```

### server mutation(서버에서의 데이터 수정)
- redirect: 특정 주소로 리다이렉트 가능
- revalidatePath: 해당 주소의 캐시를 즉시 업데이트
- revalidateTag: fetch 요청시 추가한 캐시 태그로 업데이트


### 서버 액션 사용시 주의점

1. 서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 없다.
2. 서버액션을 import하는 것 뿐 아니라 props형태로 클라이언트 컴포넌트에 넘기는 것은 가능하다.
3. 정리하자면 서버에서만 실행될 수 있는 자원은 꼭 분리해야한다.


## 11.6 그 밖의 변화
- 프로젝트 전체 라우트에서 사용할 수 있는 미들웨어 강화
- SEO 쉽게 작성할 수 있는 기능 추가
- 정적으로 내부 링크를 분석할 수 있는 기능 추가

