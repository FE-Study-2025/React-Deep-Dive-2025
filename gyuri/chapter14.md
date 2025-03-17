# 웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈

## 📍 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)

- 웹 애플리케이션에서 가장 많이 보이는 취약점 중 하나
- 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점
- 왜 위험한가?
  - script가 실행될 수 있다면 웹사이트 개발자가 할 수 있는 모든 작업을 함께 수행할 수 있으며, 쿠키를 획득해 사용자의 로그인 세션 등을 탈취하거나 사용자의 데이터를 변경하는 등 각 종 위험성이 있다.

```html
<p>사용자가 글을 작성했습니다.</p>
<script>
  alert('XSS1)
</script>
```

### 🗡️ dangerouslySetInnerHTML prop

- 특정 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법
- 왜 위험한가?

  - 인수로 받는 문자열에는 제한이 없다는 것

- 일반적으로 게시판과 같이 사용자나 관리자가 입력한 내용을 브라우저에 표시하는 용도로 사용
- dangerouslySetlnnerHTML이 인수로 받는 문자열에는 제한이 없다

```ts
const html = `<span><svg/onload=alert(origin)></span>`;

function App() {
  return <div dangerouslySetInnerHTML={{ html }} />;
}

export default App;
```

### 🗡️ useRef를 활용한 직접 삽입

- 왜 위험한가?
  - useRef를 활용하면 직접 DOM에 접근할 수 있으므로 이 DOM에 앞서와 비슷한 방식으로 innerHTML에 보안 취약점이 있는 스크립트를 삽입하면 동일한 문제가 발생

```ts
const html = "<span><svg/onload=alert(origin)></span>";

function App() {
  const divRef = useRef<HTMLDivElement>(nuU);

  useEffect(() => {
    if (divRef.current) {
      divRef.current.innerHTML = html;
    }
  });

  return <div ref={divRef} />;
}
```

이외에는

- `<a>` 태그에 잘못된 href를 삽입
- onclick, onload 이벤트

### 🛡️ 리액트에서 XSS 문제를 피하는 방법

- 제3자가 삽입할 수 있는 HTML을 안전한 HTML 코드로 한 번 치환하는 것 => `새니타이즈(sanitize)` or `이스케이프(escape)`
- 라이브러리 사용해서 구현 가능
  - [DOMpurity](https://github.com/cure53/DOMPurify)
  - [sanitize-html](https://github.com/apostrophecms/sanitize-html)
  - [js-xss](https://github.com/leizongmin/js-xss)
- 단순히 보여줄 때뿐만 아니라 사용자가 콘텐츠를 저장할 때도 한번 이스케이프 과정 을 거치는 것이 더 효율적이고 안전
- 서버에서 수행하는 것이 좋음

  - POST 요청을 스크립트나 curl 등으로 직접 요청하는 경우에는 스크립트에서 실행하는 이스케이프 과정을 생략하고 바로 저장될 가능성이 있기 때문
  - `curl`?
    - 명령줄에서 HTTP 요청을 보낼 수 있는 도구
    ```sh
    curl -X POST https://example.com/api/data \
    -H "Content-Type: application/json" \
    -d '{"name": "John Doe", "email": "john@example.com"}'
    ```

**sanitize-html**

- 허용 목록(allow list) 방식 : 허용할 태그와 목록을 일일히 나열

사용자가 콘텐츠를 저장할 때도 한번 이스케이프 과정을 거치는 것이 더 효율적이고 안전
치환 과정은 되도록 서버에서 수행하는 것이 좋다.
스크립트나 curl 등으로 직접 요청하는 경우에는 스크립트에서 실행하는 이스케이프 과정을 생략하고 바로 저장될 가능성이 있기 때문이다.

쿼리스트링에 있는 내용을 그대로 실행하거나 보여주는 경우 에도 보안 취약점이 발생할 수 있다
query, GET 파라미터，서버에 저장된 사용자가 입력한 데이터 등 외부에 존재하는 모든 코드를 위험한 코드로 간주하고 이를 적절하게 처리하는 것이 좋다.

> 리액트의JSX 데이터바인딩
> 리액트는 XSS를 방어하기 위해 이스케이프 작업이 존재한다.

```ts
const html = '<span><svg/onload=alert(origin)x/span>,

function App() {
return <div id={htnil}>{html}</div>

}
```

> `<div>{html}</div>`와 같이 HTML에 직접 표시되는 textcontent와 HTML 속성값에 대해서는 리액트가 기본적으로 이스케이프 작업을 해준다.
>
> dangerouslySetlnnerHTML이나 props로 넘겨받는 값의 경우 개발자의 활용도에 따라 원본 값이 필요할 수 있기 때문에 이러한 작업이 수행되지 않는다.

## 📍 getServerSideProps와 서버 컴포넌트를 주의하자

- getServerSideProps와 서버 컴포넌트가 클라이언트 컴포넌트에 반환되는 props는 반드시 필요한 값으로만 제한하기!
- 쿠키 전체를 넘겨주는 행위 등은 하지말기!
- 이유?
  - 반환하는 props값은 모두 사용자가 접근 가능해지기 때문

## 📍 `<a>` 태그의 값에 적절한 제한을 둬야 한다

- `<a>` 태그의 href에 `javascript:`로 시작하는 자바스크립트 코드를 넣어둔 경우
  - 기본 기능을 막고 별도 이벤트 핸들러만 작동시키기 위한 용도 => 안티패턴
  - href에 사용자가 입력한 주소를 넣을 수 있다면 보안 이슈 발생
  - href로 들어갈 수 있는 값을 제한하기 !!
    - ex. `<a href={isSafeHref(safeHref) ? safeHref : '#'}>안전한 href</a>`

## 📍 HTTP 보안 헤더 설정하기

- 브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더를 의미

### 🛡️ Strict-Transport-Security

- 모든 사이트가 HTTPS를 통해 접근해야 하며，만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.
- `Strict-Transport-Security: max-age=<expire-1ime>; includeSubDomains`
  - `<expire-time>`: 이 설정을 브라우저가 기억해야 하는 시간을 의미하며，초 단위로 기록
  - `includeSubDomains`: 모든 하위 도메인에도 규칙 적용

### 🛡️ X-XSS-Protection

- 비표준 기술로，현재 사파리와 구형 브라우저에서만 제공되는 기능
- 페이지에서 XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더

### 🛡️ X-Frame-Options

- 페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낼 수 있다.
- 사용자는 이 페이지를 진짜 네이버로 오해할 수 있고，공격자는 이를 활용해 사용자의 개인정보 를 탈취할 수 있다
- `X-Frame-Options: DENY` // 만약 위와 같은 프레임 관련 코드가 있다면 무조건 막는다.
- `X-Frame-Options: SAMEORIGIN` // 같은 origin에 대해서만 프레임을 허용

### 🛡️ Permissions-Policy

- 웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더
  - ex. 카메라나 GPS와 같이 브라우저가 제공하는 기능
- 왜 필요한가?
  - 브라우저에서 사용자의 위치를 확인하는 기능(geolocation)과 관련된 코드를 전혀 작성하지 않았다고 가정 해보자. 그러나 해당 기능이 별도로 차단돼 있지 않고，그 와중에 XSS 공격 등으로 인해 이 기능을 취득해서 사용하게 되면 사용자의 위치를 획득할 수 있게 된다.

```
# 모든 geolocation 사용을 막는다.
Permissions-Policy: geolocation=()

# geolocati에을 페이지 자신과 몇 가지 페이지에 대해서만 허용한다.
Permissions-Policy: geolocation=(self "https://a.yceffort.kr" "https://b.yceffort.kr")

# 카메라는 모든 곳에서 허용한다.
Permissions-Policy: camera=*;

# pip 기능을 막고, geolocation은 자신과 특정 페이지만 허용하며, # 카메라는 모든 곳에서 허용한다.
Permissions-Policy: picture-in-picture=(), geolocation=(self https://yceffort.kr), camera=*;
```

- [MDN 사이트](https://developer.mozilla.org/en-US/docs/Web/HTTP/Permissions_Policy#browser_compatibility%20,%20https://github.com/w3c/webappsec-%20permissions-policy/blob/main/features.md)
- [기능 선택해 헤더를 만들어주는 사이트](https://www.permissionspolicy.com/)

### 🛡️ X-Content-Type-Options

- Content-type 헤더에서 제공하는 MIME 유형이 브라우저에 의해 임의로 변경되지 않게 하는 헤더
  - 즉，Content-type: text/css 헤더가 없는 파일은 브라우저가 임의로 CSS로 사용할 수 없으며，Content-type: text/javascript나 Content-type: application/javascript 헤더가 없는 파일은 자바스크립트로 해석할 수 없다.
- 웹서버가 브라우저에 강제로 이 파일을 읽는 방식을 지정하는 것
  - MIME? Multipurpose Internet Mail Extensions의 약자로，Content-type의 값으로 사용된다. 이름에서처럼 원래는 메일을 전송할 때 사용하던 인코딩 방식으로，현재는 Content-type에서 대표적으로 사용되고 있다.
- `X-Content-Type-Options: nosniff`

### 🛡️ Referrer-Policy

- HTTP 요청의 Referer 헤더
  - 사용자가 어디서 와서 방문 중인지 인식할 수 있 는 헤더지만，반대로 사용자 입장에서는 원치 않는 정보가 노출될 위험도 존재

<img width="598" alt="image" src="https://github.com/JAVACAFE-STUDY/2024-modern-react-deep-dive-study/assets/97719273/a2a81b5d-c36e-40f4-9e23-91339577af2f">
<img width="598" alt="image" src="https://github.com/JAVACAFE-STUDY/2024-modern-react-deep-dive-study/assets/97719273/ba13c9cc-a35d-42a7-ba18-3c4ed3bc73e4">

- 구글에서는 이용자의 개인정보 보호를 위 해 strict-origin-when-cross-origin 혹은 그 이상을 명시적으로 선언해 둘 것을 권고

### 🛡️ Content-Security-Policy(콘텐츠 보안 정책)

- XSS 공격이나 데이터 삽입 공격과 같은 다양한 보안 위협을 막기 위해 설계

#### \*-src

- ex. `Content-Security-Policy: font-src <source> <source>;`
- 만약 해당 -src가 선언돼 있지 않다면 default-src로 한 번에 처리할 수도 있다. (폴백 역할)
- [사용 가능한 모든 지시문](https://www.w3.org/TR/CSP2/#directives)

#### form-action

- 폼 양식으로 제출할 수 있는 URL을 제한할 수 있음
  `<meta http-equiv="Content-Security-Policy" content="form-action 'none'" />`

### 🛡️ 보안 헤더 설정하기

- Next.js
  - 애플리케이션 보안을 위해 HTTP 경로별로 보안 헤더를 적용할 수 있음
- NGINX
  - 경로별로 `add_header` 지시자를 사용해 원하는 응답 헤더를 추가할 수 있음

### 🛡️ 보안 헤더 확인하기

- [확인할 수 있는 사이트](https://securityheaders.com/)

## 📍 OWASP Top 10

- 웹 애플리케이션에서 발생할 수 있는 주요 보안 취약점을 요약해 주는 것뿐만 아니라 이 문제를 어떻게 조치해야 하는지도 자세히 소개
- OWASP Top 10을 기준으로 자신이 운영하는 웹사이트를 살펴보고 어떠한 보안 취약점이 존재할 수 있는지，현재 문제가 되는 부분은 무엇인지 등을 한번 돌이켜보고 점검해 보자.
