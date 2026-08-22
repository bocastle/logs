# React Context 성능 최적화 정리

## 핵심 요약

- 변경 빈도가 다른 상태와 action Context를 나누면 dispatch 함수만 읽는 버튼은 데이터 변경에 다시 렌더링되지 않는다.
- useMemo로 value를 감싸도 포함된 state가 바뀌면 모든 기본 useContext 소비자는 다시 렌더링된다.
- React Profiler의 render reason으로 어떤 Provider 변경이 비싼 소비자를 깨우는지 먼저 측정한다.

## 개념 설명

React Context 최적화는 provider 값이 바뀔 때 영향을 받는 소비자 범위를 줄이는 작업이다.

Context value의 참조가 바뀌면 해당 context를 읽는 컴포넌트가 다시 렌더링되므로 상태와 액션을 나누거나 provider를 분리한다.

## 예시

```tsx
const value = useMemo(() => ({ theme, setTheme }), [theme]);
return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
```

value 참조를 안정화하고 provider 범위를 좁히면 unrelated 화면의 렌더링을 줄일 수 있다.

## 면접 답변 예시

> value 객체의 참조를 안정화하면 부모의 unrelated render가 같은 Context 소비자를 깨우는 일을 줄일 수 있다. 객체를 제자리에서 변경해 같은 참조를 넘기면 소비자가 갱신되지 않아 화면이 오래된 Context 값을 표시한다. 매우 자주 바뀌는 외부 데이터에는 Context 전체 전파 대신 useSyncExternalStore 기반 선택 구독을 검토한다.

## 장점

- Provider 범위를 실제 소비 subtree 가까이에 두면 값 갱신이 페이지의 무관한 형제 영역까지 전파되지 않는다.

## 단점

- 작은 값마다 Provider를 중첩하면 소유 관계와 테스트 setup이 복잡해져 최적화 비용이 이득을 넘을 수 있다.

## 주의사항 / 실무 팁

- 읽기 전용 state와 안정적인 명령 함수가 다른 소비 패턴이면 두 Context로 분리해 구독 범위를 맞춘다.
