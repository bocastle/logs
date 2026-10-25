# React 상태 업데이트 큐 정리

## 핵심 요약

- functional updater는 상태 업데이트 큐의 직전 계산 결과를 받아 같은 event의 연속 증가를 모두 반영한다.
- 캡처된 count로 `setState(count + 1)`을 두 번 호출하면 두 요청이 같은 값을 써서 한 번만 증가한다.
- 다음 값이 이전 값에 의존하면 항상 `setState(prev => next)` 형태로 큐 관계를 코드에 표시한다.

## 개념 설명

React 상태 업데이트 큐는 같은 렌더 사이클에 들어온 setState 요청을 순서대로 모아 다음 상태를 계산하는 구조다.

객체 값 업데이트는 캡처된 값을 사용할 수 있으므로 이전 상태에 의존하면 functional updater를 써야 한다.

## 예시

```tsx
setCount((current) => current + 1);
setCount((current) => current + 1);
```

함수형 업데이트를 쓰면 같은 이벤트 안에서 두 번 증가해도 최신 큐 상태를 기준으로 계산한다.

## 면접 답변 예시

> updater 순서가 코드 순서대로 적용되어 이전 state에 의존하는 복합 변경을 예측 가능하게 계산한다. setState 직후 같은 handler에서 state 변수를 읽으면 아직 commit되지 않은 이전 렌더 값을 보게 된다. 한 event와 Promise callback에서 여러 업데이트를 예약하는 테스트로 최종 값과 render 횟수를 확인한다.

## 장점

- React가 여러 setState를 한 render로 처리해 중간 값마다 DOM을 commit하는 비용을 줄인다.

## 단점

- updater 함수에 로그 전송 같은 부작용을 넣으면 개발 검증이나 중단된 render에서 예상보다 여러 번 실행될 수 있다.

## 주의사항 / 실무 팁

- commit된 DOM이나 state를 관찰해야 하는 작업은 effect로 옮기고 동기 읽기를 기본 흐름으로 만들지 않는다.
