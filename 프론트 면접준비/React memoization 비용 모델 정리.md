# React memoization 비용 모델 정리

## 핵심 요약

- memo가 비싼 row 렌더를 건너뛰면 큰 목록의 상위 state 변경에서 CPU 시간을 절약할 수 있다.
- shallow compare 자체가 매우 싼 컴포넌트 렌더보다 비싸면 memo를 추가한 뒤 총 작업량이 늘어난다.
- Profiler에서 commit 빈도와 actual duration을 기록한 뒤 회피 가능한 render가 큰 컴포넌트부터 최적화한다.

## 개념 설명

React memoization 비용 모델은 비교 비용, 캐시 보관 비용, 재렌더링 회피 이득을 함께 따지는 기준이다.

`memo`, `useMemo`, `useCallback`은 참조 안정성을 주지만 shallow compare와 의존성 관리 비용이 생긴다.

## 예시

```tsx
const Row = memo(function Row({ item }: { item: Item }) {
  return <li>{item.name}</li>;
});
```

큰 목록의 row처럼 props가 안정적이고 렌더링이 비싼 곳에서 이득을 측정한다.

## 면접 답변 예시

> 계산 비용과 호출 빈도가 모두 높은 selector를 캐시하면 같은 입력에서 중복 연산을 피한다. 큰 계산 결과와 dependency를 오래 보관하면 화면에서 더 쓰지 않는 데이터가 메모리에 남는다. 데이터 규모가 줄거나 렌더 비용이 낮아지면 memoization을 제거한 기준과 다시 비교해 복잡성의 이득을 재평가한다.

## 장점

- useCallback으로 안정화한 handler가 memoized child의 props 비교를 통과해 불필요한 render를 줄인다.

## 단점

- custom comparator가 callback 변경을 무시하면 자식이 오래된 closure를 호출하는 정확성 버그가 생긴다.

## 주의사항 / 실무 팁

- 매번 새로 만들어지는 object prop은 필요한 primitive로 나누거나 소유자에서 안정화해 비교 실패 원인을 줄인다.
