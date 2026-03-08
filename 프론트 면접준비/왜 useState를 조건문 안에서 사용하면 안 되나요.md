# 왜 useState를 조건문 안에서 사용하면 안 되나요?

## 핵심 요약

- `useState()`는 **호출 순서(call order)** 를 기준으로 상태를 매핑합니다.
- 조건문 안에서 `useState()`를 호출하면 렌더링마다 **훅 호출 개수/순서가 달라질 수 있어**, 리액트가 상태를 추적하는 과정에서 **불일치와 버그**가 발생합니다.
- 따라서 `useState()`는 **컴포넌트 최상위(top-level)** 에서 항상 동일한 순서로 호출되어야 합니다.

## 개요

`useState()`를 조건문 안에서 사용하면 안 되는 이유는 리액트가 `state`를 관리하는 방식이 `useState`를 호출하는 순서와 연관되어 있기 때문입니다.

리액트는 컴포넌트 내부에서 `useState()`가 호출된 **순서**를 기준으로 `state`를 저장하고 업데이트합니다. 그런데 `useState()`를 조건문 안에서 호출하면 특정 렌더링 시에는 호출되고, 다른 렌더링에서는 호출되지 않을 가능성이 생깁니다. 이렇게 되면 리액트가 호출 순서를 기반으로 `state`를 추적하는 과정에서 혼동이 발생하게 됩니다.

## 문제 예시

예를 들어, 다음과 같은 코드가 있다고 가정해 보겠습니다.

```jsx
function Example({ shouldUseState }) {
  if (shouldUseState) {
    const [count, setCount] = useState(0);
  }

  return <div>Example Component</div>;
}
```

위 코드에서 `shouldUseState` 값이 `true`일 때만 `useState()`가 호출됩니다. 문제는, `shouldUseState`가 `false`로 바뀌는 경우입니다. 이때 `useState()`의 호출 개수가 변경되면서 리액트의 내부 상태 저장 방식과 불일치가 발생합니다.

리액트는 상태를 **순서 기반으로 관리**하기 때문에, 이후 렌더링에서 이전에 있던 `useState()` 호출이 사라지면 다른 `useState()` 호출들이 엇갈려 버그가 발생할 수 있습니다.

## 해결 방법

이러한 문제를 방지하기 위해서는 `useState()`가 항상 컴포넌트의 **최상위**에서 호출되어야 합니다. 즉, 조건문이나 반복문, 함수 내부에서 호출하지 않아야 합니다.

### 안전한 패턴 예시

```jsx
function Example({ shouldUseState }) {
  // 항상 동일한 순서로 호출
  const [count, setCount] = useState(0);

  if (!shouldUseState) {
    return <div>Example Component</div>;
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
    </div>
  );
}
```

## 장점

- 훅 호출 순서가 보장되어 상태가 안정적으로 매핑되고, 예측 가능한 렌더링이 가능합니다.
- 훅 규칙을 지키면 린터(ESLint rules of hooks)와 생태계 도구의 도움을 안정적으로 받을 수 있습니다.

## 단점

- 조건에 따라 “상태 자체를 만들지 않겠다”는 설계를 그대로 표현하기 어렵고, 보통은 **상태를 항상 선언해 두고** 조건에 따라 **사용/렌더링만 분기**하게 됩니다.

## 주의사항 및 실무 팁

- 훅은 반드시 **컴포넌트 최상위**에서 호출합니다.
  - 조건문(`if`), 반복문(`for/while`), 중첩 함수/콜백 내부에서 호출하지 않습니다.
- 조건부 로직이 필요하면 훅 호출이 아니라 **렌더링 분기**(early return) 또는 **커스텀 훅으로 로직 분리**를 고려합니다.
