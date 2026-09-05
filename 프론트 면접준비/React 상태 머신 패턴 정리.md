# React 상태 머신 패턴 정리

## 핵심 요약

- 상태 머신이 허용 state와 event를 제한하면 loading과 success가 동시에 true인 조합을 만들 수 없다.
- 작은 토글까지 별도 상태와 event로 모델링하면 전이 수가 늘어 단순한 UI보다 정의가 더 어려워진다.
- 먼저 가능한 state, 허용 event, 금지 전이를 표로 작성하고 실제 제품 흐름과 맞는지 검토한다.

## 개념 설명

React 상태 머신 패턴은 화면 상태를 가능한 state와 event의 전이표로 제한하는 설계다.

boolean 여러 개 대신 `idle`, `loading`, `success`, `error` 같은 단일 상태와 허용 이벤트를 reducer나 머신으로 처리한다.

## 예시

```tsx
type ViewState = { tag: "idle" } | { tag: "loading" } | { tag: "error"; message: string };
function send(event: Event) {
  dispatch({ type: event.type });
}
```

불가능한 조합을 줄여 로딩이면서 동시에 성공인 화면 같은 모순을 막는다.

## 면접 답변 예시

> 요청 id를 event에 포함하면 늦게 도착한 성공 응답을 현재 loading 상태가 아닌 전이로 거부할 수 있다. 병렬로 움직이는 업로드와 편집 상태를 하나의 평면 state로 합치면 조합 수가 폭발한다. 모든 state-event 조합을 table-driven test로 순회해 금지 이벤트가 state를 바꾸지 않는지도 확인한다.

## 장점

- 각 전이에 이름이 있어 사용자의 재시도, 취소, 완료가 로그와 테스트에서 같은 언어로 남는다.

## 단점

- transition 함수 안에서 네트워크 요청을 실행하면 같은 event 재현이 외부 부작용 때문에 결정적이지 않다.

## 주의사항 / 실무 팁

- 순수 transition은 다음 state만 반환하고 command나 effect는 전이 결과를 관찰하는 별도 계층에서 실행한다.
