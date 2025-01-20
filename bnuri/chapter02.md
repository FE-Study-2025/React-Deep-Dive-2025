# 2. 리액트 핵심 요소 깊게 살펴보기

## 2.1 JSX

- 페이스북(현 메타)에서 소개한 새로운 구문
- 자바스크립트 표준의 일부는 아님
- 트랜스파일러를 거쳐야 자바스크립트 런타임이 이해할 수 있는 자바스립트 코드로 변환됨

### JSX의 목적
- HTML이나 XML을 자바스크립트 내부에 표현 
- 다양한 속성을 가진 트리 구조를 토큰화해 ECMAScript로 변환

즉, 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 새로운 문법

### JSX의 정의

#### JSXElement
- JSX를 구성하는 가장 기본 요소 
```xml
<JSXElement> // <- JSXOpeningElement
</JSXElement> // <- JSXClosingElement

<JSXElement /> // <- JSXSelfClosingElement

<></> // <- JSXFragment

```
##### JSXElementName
- JSXElement의 요소 이름으로 쓸 수 있는 것
- JSXIdentifier : JSX내부에서 사용할 수 있는 식별자. $, _ 가능. 숫자로 시작하거나 다른 특수문자로 시작할 수 없음
- JSXNamespacedName : JSXIdentifier:JSXIdentifier 조합. :로 묶을 수 있는 것은 한 개.
- JSXMemberExpression : JSXIdentifier.JSXIdentifier 조합. .을 여러 개 이어서 하는 것 가능. 단, :과 이어서 사용 불가.
- 
#### JSXAttributes
- JSXElement에 부여할 수 있는 속성
- JSXSpreadAttributes: 자바스크립의 전개 연산자와 동일한 역할
- JSXAttribute: 속성을 나타내는 키와 값으로 짝을 이루어 표현

#### JSXChildren
JSXElement의 자식 값. JSX는 속성을 가진 트리 구조를 나타내기 위해 만들어졌기 떄문에 부모 자식 관계를 나타낼 수 있음

### JSX는 어떻게 자바스크립트에서 변환될까?
리액트에서 JSX를 변환하는 @babel/plugin-transform-react-jsx 플러그인
```javascript
const ComponentA = <A required={true}>Hello</A>

const ComponentB = <>Hello</>

const ComponentC - (
    <div>
        <span>hello world</span>
    </div>
)
```

변환 결과
```javascript
'use strict'

var ComponentA = React.createElement(
    A,
    { required: true },
    'Hello World',
)
var ComponentB = React.createElement(React.Fragment, null, 'Hello World')
var ComponentC = React.createElement(
    'div',
    null,
    React.createElement('span', null, 'hello world'),
)
```

## 2.2 가상 DOM과 리액트 파이버
### 가상 DOM의 탄생 배경
브라우저가 웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 듦

하나의 페이지에서 모든 작업이 일어나는 SPA에서 추가 렌더링 작업은 더욱 많음

가상DOM은 웹페이지가 표시해야 할 DOM을 일단 메모리에 저장하고 리액트가 실제 변경에 대한 준비가 완료되었을 때 실제 브라우저의 DOM에 반영


### 가상 DOM을 위한 아키텍처, 리액트 파이버(React Fiber)

#### React Fiber?
- 리액트에서 관리하는 평범한 자바스크립트 객체
- 파이버 재조정자(fiber reconciler)가 파이버 관리
- fiber reconciler는 가상 DOM과 실제 DOM을 비교, 변경 사항 수집. 차이가 있을 시 변경에 관련된 정보를 가지고 있는 파이버를 기준으로 화면에 렌더링 요청
- 재조정(reconciliation) : 어떤 부분을 새롭게 렌더링해야 하는지 가상 DOM과 실제 DOM을 비교하는 작업(알고리즘)

#### React Fiber의 목표
- 작업을 작은 단위로 분할, 우선순위 매김
- 작업을 일시 중지, 나중에 다시 시작 가능
- 이전에 했던 작업을 재사용하거나 필요하지 않은 경우 폐기
- 이러한 모든 과정은 비동기로 일어남

과거 리액트 조정 알고리즘은 스택 알고리즘으로 이루어져 동기적으로 작업이 이루어짐 -> 비효율성

이를 타파하기 위해 리액트 팀은 스택 조정자 대신 파이버라는 개념을 탄생시킴

#### Fiber는 어떻게 구현되어 있을까?
- 하나의 작업 단위로 구성
- 작업 단위를 하나씩 처리 후 finishedWork() 로 작업 마무리
- 작업을 커밋 ->  실제 브라우저 DOM에 가시적인 변경 사항 만듦

1. 렌더 단계: 사용자에게 노출되지 않는 모든 비동기 작업 수행. Fiber의 우선순위 지정, 중지, 폐기 등
2. 커밋 단계: DOM에 실제 변경 사항 반영을 위한 작업. commitWork() 실행. 동기식으로 일어나며 중단될 수 없음

리액트 요소는 렌더링이 발생할 때마다 새롭게 생성되지만, Fiber는 가급적이면 재사용됨

#### React Fiber Tree
파이버 트리는 리액트 내부에 두 개 존재
1. 현재 모습을 담은 파이버 트리
2. 작업 중인 상태를 나타내는 workInProgress 트리

리액트 파이버의 작업이 끝나면 리액트는 단순히 포인터만 변경해 workInProgress 트리를 현재 트리로 변경 (더블 버퍼링)

#### Fiber의 작업 순서
1. 리액트는 beginWork() 함수를 실행하여 파이버 작업 수행. 더 이상 자식이 없는 파이버를 만날 때까지 트리 형식으로 시작
2. 1번에서 작업이 끝난다면 그다음 completeWork() 함수를 실행해 파이버 작업을 완료
3. 형제가 있다면 형제로 넘어감
4. 2번, 3번이 모두 끝나다면 return으로 돌아가 자신의 작업이 완료됐음을 알림

생성된 트리에서 setState 등으로 업데이트가 발생한다면?

workInProgress 트리를 다시 빌드하기 시작. 이제는 파이버가 이미 존재하므로 되도록 새로 생성하지 않고 기존 파이버에서 업데이트된 props를 받아 파이버 내부에서 처리

>가상 DOM과 리액트의 핵심은 브라우저의 DOM을 더욱 빠르게 그리고 반영하는 것이 아니라, 값으로 UI를 표현하는 것
> 
>화면에 표시되는 UI를 자바스크립트의 문자열, 배열 등과 마찬가지로 값으로 관리하고 이러한 흐름을 효율적으로 관리하기 위한 메커니즘이 바로 리액트의 핵심


## 2.3 클래스형 컴포넌트와 함수형 컴포넌트

## 2.4 렌더링은 어떻게 일어나는가?

### 리액트의 렌더링이란?
리액트 애플리케이션 트리 안에 있는 모든 컴포너트들이 현재 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정

### 리액트의 렌더링이 일어나는 이유
1. 최초 렌더링
2. 리렌더링
   1. 클래스형 컴포넌트의 setState가 실행되는 경우
   2. 클래스형 컴포넌트의 forceUpdate가 실행되는 경우
   3. 함수형 컴포넌트의 useState()의 두 번재 배열 요소인 setter가 실행되는 경우
   4. 함수형 컴포넌트의 useReducer()의 두 번째 배열 요소인 dispatch가 실행되는 경우
   5. 컴포넌트의 key props가 변경되는 경우
   6. props가 변경되는 경우
   7. 부모 컴포넌트가 렌더링될 경우
     **(부모 컴포넌트가 리렌더링된다면 자식 컴포넌트도 무조건 리렌더링이 일어난다)**

### 리액트의 렌더링 프로세스
1. 컴포넌트의 루트에서부터 차근차근 아래쪽으로 내려가면서 업데이트가 필요하다고 지정돼 있는 모든 컴포넌트를 찾음
2. 클래스형 컴포넌트의 경우에는 클래스 내부의 render() 함수 실행, 함수형 컴포넌트의 경우에는 FunctionComponent() 그 자체를 호출한 뒤 결과를 저장
3. 각 컴포넌트의 렌더링 결과물을 수집
4. 리액트의 새로운 트리인 가상 DOM과 비교해 실제 DOM에 반영하기 위한 모든 변경 사항을 차례로 수집 (여기 까지가 리액트의 재조정 Reconciliation)
5. 재조정 과정이 모두 끝나면 모든 변경 사항을 하나의 동기 시퀀스로 DOM에 적용

### 렌더와 커밋
#### 렌더 단계
- 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업
- 렌더링 프로세스에서 컴포넌트를 실행해 이 결과과 이전 가상 DOM을 비교하는 과정을 거쳐 변경이 필요한 컴포넌트를 체크하는 단계
- 여기서 비교하는 것은 크게 type, props, key

#### 커밋 단계
- 렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정
- 이 단계가 끝나야 비로소 브라우저의 렌더링 발생
- 리액트가 DOM을 커밋 단계에서 업데이트 한다면 이렇게 만들어진 모든 DOM 노드 및 인스턴스를 가리키도록 리액트 내부의 참조를 업데이트
- 그다음 생명주기 개념이 있는 클래스형 컴포넌트에서는 componentDidMount, componentDidUpdate 메서드 호출, 함수형 컴포넌트에서는 useLayoutEffect 훅 호출

> 리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는 것은 아니다. 렌더링을 수행 했으나 커밋 단계까지 갈 필요가 없다면 커밋 단계는 생략될 수 있다.

## 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

섣부른 최적화 (premature optimization or premature memoization)를 경계하자

허나 일반적으로 props에 대한 얕은 비교를 수행하는 것보다 리액트 컴포넌트의 결과물을 다시 계산하고 실제 DOM까지 비교하는 작업이 더 무겁고 비싸다. 
따라서 조금이라도 로직이 들어간 컴포넌트는 메모이제이션이 성능 향상에 도움을 줄 가능성이 크다.

성능에 대해 지속적으로 모니터링하고 관찰하는 것보다 섣부른 메모이제이션 최적화가 주는 이점이 더 클 수도 있다.