# User Timing API 정리

## 핵심 요약

- 제품 흐름별 병목을 같은 이름으로 추적할 수 있다.
- mark 이름 규칙이 없으면 분석 축이 흩어진다.
- route, action, phase 순서로 mark 이름을 정한다.

## 개념 설명

User Timing API는 제품 코드가 직접 성능 마커와 측정 구간을 남기게 해 주는 브라우저 성능 API다.

`performance.mark()`로 시작과 종료 지점을 찍고 `performance.measure()`로 두 mark 사이 duration을 만든다. 브라우저 기본 지표가 모르는 제품 흐름을 이름 붙일 때 쓴다.

## 예시

```js
performance.mark("filter-open-start");
await renderFilterPanel();
performance.mark("filter-open-end");
performance.measure("filter-open", "filter-open-start", "filter-open-end");
```

필터 패널 열림처럼 제품에 중요한 구간을 `performance.mark()`와 `performance.measure()`로 직접 측정한다.

## 면접 답변 예시

> User Timing API는 browser가 자동으로 모르는 제품 흐름에 start와 end mark를 직접 남겨 duration을 재는 기능입니다. 예를 들어 filter panel이 열리는 시간을 같은 이름으로 기록하면 배포 전후와 device별 차이를 비교할 수 있습니다. 다만 mark 이름에 사용자 값이나 고유 ID를 넣으면 cardinality와 개인정보 문제가 생기므로 route·action·phase 규칙으로 제한하겠습니다. 측정 뒤 entry를 정리하고 Web Vitals와 함께 보내 해당 기능 지연이 실제 사용자 경험에도 영향을 줬는지 확인합니다.

## 장점

- DevTools trace와 RUM 데이터를 같은 mark 이름으로 연결한다.

## 단점

- 민감한 사용자 입력을 mark 이름에 넣으면 로그 위험이 생긴다.

## 주의사항 / 실무 팁

- `clearMarks()`와 `clearMeasures()` 정책을 둔다.
