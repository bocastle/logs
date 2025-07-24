# 왜 `useState`를 조건문 안에서 사용하면 안 되나요?

리액트에서 `useState()`를 조건문 안에서 사용하면 안 되는 이유는 **리액트가 state를 관리하는 방식이 `useState()` 호출의 순서에 의존**하기 때문입니다.

---

## 리액트의 state 관리 방식

리액트는 훅(hook)의 호출 순서를 기준으로 각 상태를 추적합니다.  
즉, `useState`를 몇 번째로 호출했는지를 기억하고, 다음 렌더링 시에도 같은 순서로 호출되기를 기대합니다.

---

## 조건문 안에서 사용할 경우의 문제

```jsx
function Example({ shouldUseState }) {
  if (shouldUseState) {
    const [count, setCount] = useState(0); // ❌ 위험한 패턴
  }

  return <div>Example Component</div>;
}
```

### 문제점:

- `shouldUseState`가 `true`일 때만 `useState`가 호출됩니다.
- 만약 다음 렌더링에서 `shouldUseState`가 `false`로 바뀌면,
  - 이전에 호출됐던 `useState()`가 **호출되지 않게 되므로**
  - 이후의 훅들(`useEffect`, 다른 `useState`)의 순서가 어긋나고,
  - 리액트는 어떤 상태가 어떤 훅에 해당하는지 **헷갈리게 됩니다.**

---

## 권장 방식: 항상 최상위에서 호출

```jsx
function Example({ shouldUseState }) {
  const [count, setCount] = useState(0); // ✅ 항상 호출됨

  return <div>{shouldUseState ? `Count: ${count}` : "State 미사용 중"}</div>;
}
```

- 훅은 항상 컴포넌트의 **최상위(top level)** 에서 호출해야 합니다.
- `조건문`, `반복문`, `중첩된 함수` 등 내부에서 훅을 호출하지 마세요.

---

## 정리

| 잘못된 사용 예                   | 올바른 사용 예                          |
| -------------------------------- | --------------------------------------- |
| `if`, `for`, `while` 안에서 호출 | 컴포넌트 최상단에서 항상 호출           |
| 렌더링마다 호출 수 달라짐        | 호출 순서가 일관됨 (리액트가 추적 가능) |

---

## 참고

- [React 공식 문서 - Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
