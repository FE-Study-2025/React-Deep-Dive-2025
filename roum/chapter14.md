# 웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈

- 보안 이슈는 버그와 같이 개발자의 실수에서 비롯되는 경우도 있지만 이러한 코드나 프로세스가 보안에 문제가 된다는 것을 인지하지 못하는 경우에도 발생한다.

## 리액트에서 발생하는 크로스 사이트 스크립팅 (XSS)

- dangerouslySetInnerHTML
- useRef를 이용한 직접 삽입

### XSS 문제를 피하는 방법

- 제3자가 삽입할 수 있는 HTML을 안전한 HTML 코드로 한 번 치환하는 것

  - DOMpurity
  - sanitize-html
  - js-xss

- 서버는 '클라이언트에서 사용자가 입력한 데이터는 일단 의심한다'라는 자세로 클라이언트의 POST 요청에 있는 HTML을 이스케이프하는 것이 제일 안전

- 리액트의 JSX 데이터 바인딩 시에는 XSS를 방어하기 위한 이스케이프 작업이 존재함

## getServerSideProps와 서버 컴포넌트를 주의

- getServerSideProps가 반환하는 props 값은 모두 사용자의 HTML에 기록되고, 전역 변수로 등록됨

> https://developer.mozilla.org/en-US/docs/Web/API/atob

```js
const encodedData = btoa("Hello, world"); // encode a string
const decodedData = atob(encodedData); // decode the string
```

## <a> 태그의 값

- `<a href="javascript:;">` 사용하지 않기
  - href에 사용자가 입력한 주소를 넣을 수 있다면 보안 이슈로 이어지기 때문
  - 버튼을 쓰는 것이 마크업 관점에서도 좋음

## HTTP 보안 헤더 설정하기

- Strict-Transport-Security

  - HTTPS를 사용하도록 강제하는 헤더

- X-Frame-Options

  - 페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지
  - SAMEORIGIN을 통해 같은 origin에 대해서만 허용할 수 있음

- Permissions-Policy
  - 브라우저가 기능을 제어할 수 있도록 하는 헤더
  - geolocation, microphone, camera, fullscreen, payment 등
  - https://www.permissionspolicy.io/

> MIME?
> Multipurpose Internet Mail Extensions
> Content-type의 값으로 사용됨
>
> 이름처럼 원래 메일에서 사용하던 인코딩 방식, 현재는 Content-type에서 대표적으로 사용

- Referrer-Policy
  - HTTP 요청에는 Referer라는 헤더가 존재
    - 현재 요청을 보낸 페이지의 주소를 나타냄
    - Referer는 오타이나, 이미 표준으로 등록된 이후에 오타임을 발견해서 계속 사용중
  - Referrer-Policy 헤더는 이러한 Referer 헤더에 사용할 수 있는 데이터를 나타냄
  - 구글에서는 이용자의 개인정보 보호를 위해 strict-origin-when-cross-origin 혹은 그 이상을 권장

## 보안 헤더 설정하기

- https://nextjs.org/docs/advanced-features/security-headers

## 보안 헤더 확인하기

- https://securityheaders.com/

## 취약점이 있는 패키지 사용 피하기

- lock.json의 모든 의존성을 파악하는 것은 불가능
