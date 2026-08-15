# Navigation Timing API 정리

## 핵심 요약

- 초기 페이지 로딩 병목을 큰 구간으로 나눌 수 있다.
- SPA 내부 라우팅은 navigation entry 하나로 설명되지 않는다.
- LCP, INP 같은 사용자 지표와 함께 본다.

## 개념 설명

Navigation Timing API는 문서 요청부터 로드 이벤트까지의 페이지 내비게이션 구간을 측정하는 성능 API다.

`PerformanceNavigationTiming`의 `domInteractive`, `responseStart`, `loadEventEnd`를 보면 서버 응답, HTML 파싱, 하위 리소스 로드의 대략적 흐름을 나눌 수 있다.

## 예시

```js
const nav = performance.getEntriesByType("navigation")[0];
report({ ttfb: nav.responseStart - nav.requestStart, domInteractive: nav.domInteractive });
```

초기 문서 로딩에서 서버 대기와 DOM interactive 시점을 분리해 기록한다.

## 면접 답변 예시

> Navigation Timing API는 document navigation의 request, response와 DOM·load 구간을 `PerformanceNavigationTiming`으로 측정합니다. `responseStart - requestStart`로 문서 TTFB를 보고 `domInteractive`와 load event를 나눠 server 대기와 parsing·resource 구간을 좁힐 수 있습니다. Load event가 사용자 체감 완료와 같지는 않으므로 LCP와 INP를 함께 봅니다. SPA 내부 route와 bfcache 복귀는 별도 navigation 유형과 application timing으로 측정하겠습니다.

## 장점

- SSR 응답 지연과 클라이언트 파싱 지연을 구분한다.

## 단점

- bfcache 복귀는 일반 navigation과 다르게 해석해야 한다.

## 주의사항 / 실무 팁

- `type`이 reload, navigate, back_forward인지 구분한다.
