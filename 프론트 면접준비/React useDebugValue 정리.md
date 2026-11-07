# React useDebugValue 정리

## 핵심 요약

- useDebugValue로 구독 상태를 표시하면 DevTools에서 여러 Custom Hook 인스턴스의 연결 여부를 빠르게 구분한다.
- formatter 밖에서 큰 객체를 직렬화하면 DevTools를 열지 않아도 매 렌더 비용을 지불한다.
- 연결 상태나 cache key처럼 진단 가치가 높은 요약값만 useDebugValue에 전달한다.

## 개념 설명

`useDebugValue`는 커스텀 훅의 내부 상태를 React DevTools에 읽기 좋은 라벨로 표시하는 훅이다.

렌더링 동작을 바꾸지 않으며 복잡한 포맷팅은 지연 formatter로 넘겨 DevTools에서 볼 때만 계산하게 한다.

## 예시

```tsx
function useOnlineStatus() {
  const online = useSyncExternalStore(subscribe, getSnapshot);
  useDebugValue(online ? "online" : "offline");
  return online;
}
```

커스텀 훅을 사용하는 컴포넌트에서 현재 구독 상태를 DevTools로 빠르게 확인할 수 있다.

## 면접 답변 예시

> 화면에 진단용 DOM을 추가하지 않고 hook 내부의 domain state를 개발 도구에 노출할 수 있다. 작은 내부 Hook마다 값을 추가하면 DevTools 목록이 소음으로 가득해 실제 문제 상태를 찾기 어려워진다. 동일 hook 인스턴스를 여러 상태로 렌더해 DevTools label이 문제를 구분하는 데 실제로 도움이 되는지 확인한다.

## 장점

- 두 번째 formatter를 사용하면 DevTools가 값을 표시할 때만 복잡한 label을 계산한다.

## 단점

- 토큰이나 사용자 원문을 debug label에 넣으면 화면 공유와 개발자 도구 캡처에 민감한 정보가 남는다.

## 주의사항 / 실무 팁

- 비싼 포맷은 `(value) => label` 인자로 넘기고 hook 동작이 formatter 결과에 의존하지 않게 한다.
