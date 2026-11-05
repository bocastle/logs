# React render props 패턴 정리

## 핵심 요약

- render props는 위치 계산이나 선택 state를 재사용하면서 호출자가 실제 DOM과 스타일을 자유롭게 결정하게 한다.
- 여러 render props를 중첩하면 들여쓰기와 control flow가 깊어져 오류 및 loading 조합을 읽기 어렵다.
- children 함수 시그니처를 구체 타입으로 선언하고 nullable 상태와 제공 action을 문서화한다.

## 개념 설명

render props 패턴은 컴포넌트가 렌더링할 내용을 함수로 받아 내부 상태나 동작을 외부 UI에 주입하는 방식이다.

상태 로직은 재사용하면서 마크업은 호출자가 결정하므로 headless 컴포넌트 설계에 자주 쓰인다.

## 예시

```tsx
<MouseTracker>{({ x, y }) => <Tooltip x={x} y={y} />}</MouseTracker>
```

위치 추적 로직과 Tooltip UI를 분리하지만 함수 children의 참조 변화는 memoization에 영향을 줄 수 있다.

## 면접 답변 예시

> children 함수의 lexical scope에서 필요한 데이터에 접근하므로 중간 wrapper로 prop을 계속 전달하지 않아도 된다. 접근성 속성까지 전부 호출자에게 맡기면 headless 로직이 제공해야 할 keyboard 계약이 구현마다 달라진다. focus 관리와 ARIA처럼 동작의 필수 조건은 getter props나 전용 component로 제공해 소비자가 빠뜨리지 않게 한다.

## 장점

- 함수 인자로 loading, error, actions를 명시해 headless 로직이 UI에 제공하는 값이 한눈에 드러난다.

## 단점

- 호출자가 매 render 새 함수를 전달하면 memoized provider의 props 비교가 항상 실패할 수 있다.

## 주의사항 / 실무 팁

- provider가 실제로 memoized되어 함수 정체성이 병목일 때만 useCallback을 적용해 불필요한 dependency를 만들지 않는다.
