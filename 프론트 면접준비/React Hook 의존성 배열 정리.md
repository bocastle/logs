# React Hook 의존성 배열 정리

## 핵심 요약

- 의존성 배열에 effect가 읽는 렌더 값을 모두 적으면 값 변경과 외부 동기화 시점이 일치한다.
- count를 배열에서 빼면 callback이 오래된 값을 잡는 stale closure 때문에 최신 화면과 다른 작업을 수행한다.
- exhaustive-deps를 비활성화하기 전에 effect를 작은 동기화 단위로 나누고 파생 값은 렌더 중 계산한다.

## 개념 설명

Hook 의존성 배열은 effect나 memo가 어떤 렌더링 값에 의존하는지 React에 알려주는 목록이다.

의존성이 빠지면 stale closure가 생기고, 불안정한 객체를 넣으면 effect가 필요 이상으로 반복된다.

## 예시

```tsx
useEffect(() => {
  document.title = `${count} unread`;
}, [count]);
```

count가 바뀔 때만 제목을 갱신해 오래된 값과 무한 반복을 피한다.

## 면접 답변 예시

> effect가 꼭 필요한 입력만 가지면 unrelated 렌더에서 구독을 해제하고 다시 연결하는 비용을 줄인다. lint 경고를 없애려고 값을 ref로 옮기면 동기화가 필요했던 변경까지 React의 추적 밖으로 숨길 수 있다. 이전 state만 필요한 갱신에는 functional updater를 사용해 읽지 않는 state를 dependency에서 자연스럽게 제거한다.

## 장점

- exhaustive-deps 검사는 closure에 포획된 props와 state의 누락을 코드 리뷰 전에 발견한다.

## 단점

- 렌더마다 만든 객체나 함수를 dependency에 넣으면 참조가 계속 바뀌어 effect가 무한 재실행될 수 있다.

## 주의사항 / 실무 팁

- effect 전용 객체는 effect 안에서 만들고 외부에 공유해야 하는 함수만 useCallback으로 정체성을 안정화한다.
