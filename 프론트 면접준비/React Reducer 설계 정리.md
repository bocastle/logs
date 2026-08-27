# React Reducer 설계 정리

## 핵심 요약

- reducer가 action과 이전 state만으로 다음 값을 계산하면 전이 규칙을 렌더러 없이 단위 테스트할 수 있다.
- reducer 안에서 fetch나 localStorage 쓰기를 실행하면 재실행 가능한 순수 계산이라는 전제가 깨진다.
- action은 판별 가능한 union으로 선언하고 각 variant가 필요한 payload만 갖도록 모델링한다.

## 개념 설명

React Reducer는 여러 상태 변경을 이벤트와 전이 규칙으로 모아 예측 가능한 상태 모델을 만드는 방식이다.

dispatch된 action type에 따라 순수 함수가 다음 state를 반환하므로 비동기 결과도 성공, 실패, 취소 이벤트로 표현한다.

## 예시

```tsx
type Action = { type: "submit" } | { type: "success"; id: string } | { type: "fail"; message: string };
function reducer(state: State, action: Action): State {
  if (action.type === "success") return { status: "saved", id: action.id };
  return state;
}
```

상태 변경 이유가 action에 남아 테스트와 회귀 분석이 쉬워진다.

## 면접 답변 예시

> 서로 의존하는 여러 필드를 한 전이에서 갱신해 중간에 불가능한 상태가 렌더링되는 일을 막는다. 서로 무관한 화면 도메인을 하나의 거대 reducer에 모으면 작은 변경도 전체 상태 구조를 알아야 한다. 비동기 작업은 effect나 event handler에서 수행하고 reducer에는 시작, 성공, 실패 결과만 dispatch한다.

## 장점

- 사용자 의도가 action 이름으로 남아 동일한 필드 변경이라도 입력, 저장 성공, 취소 원인을 구분할 수 있다.

## 단점

- action을 `{ type: string; payload: any }`로 넓히면 분기별 payload 계약과 exhaustive 검사가 사라진다.

## 주의사항 / 실무 팁

- 모든 전이에 대해 입력 state, event, 예상 state를 표로 만들고 불변식이 유지되는지 테스트한다.
