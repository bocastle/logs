# Browser Rendering Critical Path 재점검 정리

## 핵심 요약

- 첫 paint가 느린 원인을 네트워크·parser·style·layout·paint로 나눈다.
- 모든 CSS를 inline하면 HTML이 커지고 cache 재사용을 잃는다.
- cold cache와 중간 성능 mobile CPU에서 trace를 수집한다.

## 개념 설명

Browser Rendering Critical Path는 HTML에서 DOM, CSS에서 CSSOM을 만든 뒤 render tree·layout·paint·composite를 거쳐 첫 화면을 표시하는 핵심 경로다.

render-blocking stylesheet은 CSSOM 준비를 지연시키고 parser-blocking script는 DOM 구성을 멈출 수 있다. 핵심 CSS·font·LCP resource의 discovery와 main-thread style/layout 비용을 waterfall과 trace에서 함께 본다.

## 예시

```text
HTML -> DOM ---------\
                     render tree -> layout -> paint -> composite
CSS  -> CSSOM -------/
```

첫 화면의 DOM·CSSOM 준비와 LCP image request를 나란히 보아 네트워크 지연과 rendering 지연을 구분한다.

## 면접 답변 예시

> Rendering critical path는 HTML과 CSS로 DOM·CSSOM을 만든 뒤 render tree, layout, paint와 composite를 거쳐 첫 화면을 표시하는 과정입니다. 첫 paint가 늦을 때 network waterfall만 보면 main thread의 style·layout 비용을 놓치고 trace만 보면 LCP resource 발견 지연을 놓칠 수 있습니다. Parser-blocking script와 render-blocking CSS, font와 LCP image initiator를 같은 timeline에서 보겠습니다. 수정 후 cold cache와 중간급 mobile 조건에서 FCP, LCP, main-thread time과 transfer byte를 함께 비교합니다.

## 장점

- critical CSS와 비핵심 CSS의 우선순위를 정한다.

## 단점

- async·defer의 차이를 무시하면 script 실행 순서가 어긋난다.

## 주의사항 / 실무 팁

- render-blocking resource와 LCP initiator를 먼저 표시한다.
