# 13. 웹페이지의 성능을 측정하는 다양한 방법

## 13.1 애플리케이션에서 확인하기

### create-react-app
reportWebVitals: web-vitals 라이브러리로 누적 레이아웃 이동(CLS), 최초 입력 지연(FID), 최초 콘텐츠풀 페인트(FCP), 최대 콘텐츠풀 페인팅(LCP), 첫 바이트까지의 시간(TTFB)을 측정 가능

### create-next-app
NextWebVitalsMetric 제공

```javascript
// pages/_app.tsx
import { AppProps } from 'next/app';
import { reportWebVitals, NextWebVitalsMetric } from 'next-web-vitals';

function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}

export function reportWebVitals(metric: NextWebVitalsMetric) {
  console.log(metric);
}

export default MyApp;
```
핵심 웹 지표 외에도 Next.js에 특화된 사용자 지표 제공
- Next.js-hydration: 페이지가 서버사이드에서 렌더링되어 하이드레이션하는데 걸린 시간
- Next.js-route-change-to-render: 페이지가 경로를 변경한 후 페이지를 렌더링을 시작하는 데 걸리는 시간
- Next.js-render: 경로 변경이 완료된 후 페이지를 렌더링하는 데 걸린 시간
=> 지표를 살펴보며 서버 사이드 렌더링 시 오래걸리지 않는지, 페이지 전환 시 호출되는 getServerSideProps가 오래걸리지 않는지 체크

  
## 13.2 구글 라이트하우스
웹 페이지의 성능, 접근성, SEO, 최적화 등을 평가하는 도구

### 탐색모드
페이지에 접속했을 때부터  페이지 로딩이 완료될 때까지의 성능을 측정
- 성능, 접근성, 권장사항, SEO 

### 기간모드
실제 웹페이지를 탐색하는 동안 지표 측정
- 흔적 (View Tracer) : 시간의 흐름에 따라 어떻게 웹페이지가 로딩됐는지 보여줌
- 트리맵 : 페이지 불러올 때 함께 로딩한 모든 리소스를 모아 확인 가능. JS 리소스 중 어떠한 파일이 전체 데이터 로딩 중 어느 정도를 차지했는지 비율로 확인 가능


## 13.3 WebPageTest
- 웹사이트 성능 분석할 수 있는 유료 도구
- 미국, 인도, 캐나다, 독일 등 한국과 어느 정도 거리가 먼 서버를 기준으로 테스트하기 때문에 성능 지표가 안좋게 나올 수 있음

## 13.4 크롬 개발자 도구
시크릿 모드에서 성능 측정하자 (일반 모드에서는 크롬 확장 프로그램 때문에 성능 이슈를 파악하는 데 방해됨)
