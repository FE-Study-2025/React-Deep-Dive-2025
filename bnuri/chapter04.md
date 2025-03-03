# 4. 서버 사이드 렌더링

## 4.2 서버 사이드 렌더링을 위한 리액트 API 살펴보기

### renderToString
- 인수로 넘겨받은 리액트 컴포넌트를 렌더링해 HTML 문자열로 변환하는 함수
- 최초의 페이지를 HTML로 렌더링할 때 사용됨
- 이벤트 핸들러는 결과물에 포함시키지 않음 즉, JS를 HTML에 hydration 하지 않음
- 가장 상단 div태그에 data-reactroot 가 속성으로 존재함 (리액트의 루트 식별)

### renderToStaticMarkup
- rednerToSTring과 동일한 역할을 하며, 한가지 차이점이 있음
- data-reactroot가 없는 순수한 HTML로 만들어짐
- renderToSTring보다 살짝 빠름

### renderToNodeStream
- 브라우저에서 사용 불가
- 결과물을 Node.js의 ReadableStream으로 반환함
- 큰 데이터를 청크로 분할해서, 렌더링 할 때 조금씩 먼저 보여주는 역할을 함
- 대부분 서버 사이드 렌더링은 renderToNodeStream 방식을 사용함

### renderToStaticNodeStream
- renderToNodeStream과 역할은 동일함
- JS와 리액트 속성을 제외한 순수 HTML 결과물을 반환함

### hydrate
- 정적으로 생성된 HTML에 이벤트와 핸들러를 붙이는 과정
- render과 hydrate 차이점
  - hydrate는 기본적으로 렌더링된 HTML이 있다는 가정하에 수행
  - render은 빈 HTML에 react 컴포넌트를 삽입하는 형식으로 수행