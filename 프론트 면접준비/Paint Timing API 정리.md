# Paint Timing API 정리

## 핵심 요약

- 초기 흰 화면 시간을 사용자 지표로 남길 수 있다.
- paint entry만으로 느린 리소스 원인은 알기 어렵다.
- FCP를 LCP, Navigation Timing과 함께 분석한다.

## 개념 설명

Paint Timing API는 문서가 처음 그려진 시점과 첫 콘텐츠가 화면에 나타난 시점을 성능 entry로 제공하는 API다.

`paint` entry의 `first-paint`와 `first-contentful-paint`를 읽어 빈 화면 시간이 얼마나 길었는지 기록한다. LCP나 INP와 달리 초기 표시 시작점을 보는 지표다.

## 예시

```js
const paints = performance.getEntriesByType("paint");
const fcp = paints.find((entry) => entry.name === "first-contentful-paint");
report({ firstPaint: paints[0]?.startTime, fcp: fcp?.startTime });
```

`first-paint`와 `first-contentful-paint`를 RUM에 보내 초기 빈 화면 시간을 추적하는 예다.

## 면접 답변 예시

> Paint Timing API는 navigation 이후 첫 paint와 first contentful paint 시점을 performance entry로 제공합니다. FCP가 늦으면 사용자가 빈 화면을 오래 본다는 뜻이지만, 이 값만으로 CSS·font·server 중 어느 구간이 원인인지는 알 수 없습니다. Navigation Timing, resource timing과 LCP를 함께 봐서 응답 전 지연인지 첫 content 이후의 큰 element 지연인지 나누겠습니다. RUM에서는 초기 entry를 놓치지 않도록 buffered observer를 쓰고 route, device class, connection 정보를 함께 기록합니다.

## 장점

- CSS, 폰트, HTML 파싱 지연을 첫 표시 관점으로 볼 수 있다.

## 단점

- SPA route 전환에는 문서 paint entry가 새로 생기지 않는다.

## 주의사항 / 실무 팁

- `buffered` PerformanceObserver로 초기 entry 누락을 막는다.
