# Layout Shift 원인을 컴포넌트 단위로 좁히는 법 정리

## 핵심 요약

- Layout shift entry의 value와 source node를 route·component 정보에 연결해 움직인 영역을 좁힌다.
- CLS는 최근 사용자 입력과 연관된 shift를 제외하므로 entry의 `hadRecentInput`을 함께 확인한다.
- 이미지·iframe·광고 슬롯의 공간 예약과 웹폰트 전환을 먼저 점검한다.

## 개념 설명

Layout Shift는 사용자가 의도하지 않았는데 보이던 요소의 위치가 바뀌는 현상이다. CLS는 session window 안의 예상치 못한 shift를 합쳐 시각적 안정성을 나타낸다.

`layout-shift` entry에는 score와 최근 입력 여부, 가능한 경우 source node가 들어 있다. RUM에서 route와 component 식별자를 붙이면 이미지 크기 누락, 늦게 삽입된 banner, 광고와 font swap 중 어느 소유 영역이 움직였는지 찾기 쉽다. Source는 움직인 결과를 가리킬 수 있어 원인을 DOM 조상과 network timing까지 따라가야 한다.

## 예시

```js
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) console.log(entry.value, entry.sources);
}).observe({ type: "layout-shift", buffered: true });
```

실제 수집에서는 `entry.hadRecentInput`이 false인 shift를 CLS 분석 대상으로 보고 source node의 안전한 식별자를 보낸다. DOM 전체나 사용자 텍스트를 telemetry에 넣지 않는다.

## 면접 답변 예시

> Layout shift를 좁힐 때는 `layout-shift` entry의 score와 source를 route·component 정보에 연결하겠습니다. 최근 사용자 입력 뒤의 이동은 CLS에서 제외될 수 있으므로 `hadRecentInput`도 함께 확인합니다. 먼저 이미지와 iframe 크기, 광고 슬롯 공간과 font swap을 점검하고 source node만 원인이라고 단정하지 않습니다. 실제 광고와 font 조건이 있는 RUM 데이터로 개선 전후 CLS를 비교합니다.

## 장점

- Shift를 route와 component 단위로 묶어 문제의 소유권과 수정 우선순위를 나누기 쉽다.
- 실제 사용자 환경의 광고·font·동적 콘텐츠 조건을 반영할 수 있다.

## 단점

- Source node가 익명 구조이거나 움직인 결과만 가리키면 실제 원인 매핑이 어렵다.
- DOM 식별자를 과도하게 수집하면 telemetry에 사용자 데이터가 섞일 수 있다.

## 주의사항 / 실무 팁

- 이미지·iframe 크기와 web font fallback 차이를 우선 확인한다.
- Source 식별자는 안전한 component 이름으로 정규화하고 사용자 텍스트를 보내지 않는다.
