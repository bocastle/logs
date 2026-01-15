# React: `props`와 `state` 정리

> 본 문서는 React 18 기준으로 작성되었습니다.

## 한눈에 보기

| 항목      | 소유자             | 변경 가능 주체                     | 생애주기                             | 용도                                               | 대표 API                 |
| --------- | ------------------ | ---------------------------------- | ------------------------------------ | -------------------------------------------------- | ------------------------ |
| **props** | 부모(호출자)       | 부모만 변경 가능(자식은 읽기 전용) | 컴포넌트가 마운트·업데이트될 때 전달 | **외부에서 주입되는 입력값**                       | JSX 속성 전달, 컴포지션  |
| **state** | 해당 컴포넌트 자신 | 자신(또는 훅/컨텍스트 통해 위임)   | 컴포넌트 생명주기 동안 보존          | **내부에서 변하는 값(상호작용, 네트워크 응답 등)** | `useState`, `useReducer` |

---

## props

- **정의**: 부모 컴포넌트가 자식 컴포넌트로 전달하는 **읽기 전용 데이터**입니다.
- **특징**
  - **불변(immutable)**: 자식은 props를 **직접 수정하지 않습니다**.
  - **단방향 데이터 흐름**: 데이터는 **부모 → 자식**으로만 흘러 예측 가능한 렌더링을 보장합니다.
- **간단 예시**

  ```jsx
  function Child({ name }) {
    return <div>{name}</div>;
  }

  function Parent() {
    return <Child name="Alice" />;
  }
  ```

- **안티패턴**
  ```jsx
  function Child(props) {
    props.name = "New Name"; // X: 읽기 전용 위반, 버그 유발
    return <div>{props.name}</div>;
  }
  ```

### 왜 자식에서 props를 바꾸면 안 되나요?

- **단방향 데이터 흐름 원칙** 때문입니다. 변경 지점을 부모(또는 상위 상태 소유자)에 **집중**시켜 상태 추적, 디버깅, 테스트를 단순화합니다.

---

## state

- **정의**: 컴포넌트 내부에서 관리되는 **가변 데이터**로, 변경 시 **렌더링을 유발**합니다.
- **사용 시점**
  - 사용자 입력, 토글, 모달 열림/닫힘
  - 비동기 응답 데이터(로딩/에러/결과)
  - 뷰 레벨의 UI 상태(탭 선택, 페이지네이션 등)
- **간단 예시**

  ```jsx
  import { useState } from "react";

  function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
  }
  ```

---

## 자식이 값을 “바꿔야” 할 때: 상태 끌어올리기(Lifting State Up)

- **원칙**: 값을 **소유한 곳**(상태를 가진 컴포넌트)이 변경을 수행해야 합니다.
- **방법**: 부모가 상태와 **업데이트 함수**를 소유하고, 자식은 **콜백**을 통해 의도를 부모에게 알립니다.

```jsx
function Parent() {
  const [name, setName] = useState("Alice");
  return <Child name={name} onChangeName={setName} />;
}

function Child({ name, onChangeName }) {
  return (
    <div>
      <div>{name}</div>
      <button onClick={() => onChangeName("Bob")}>Change</button>
    </div>
  );
}
```

> 데이터 흐름은 여전히 **부모 → 자식**, 변경 요청은 **자식 → 부모(콜백)** 경로를 사용합니다.

---

## 파생 상태(Derived State)와 메모화

- **파생 상태는 가능하면 저장하지 말고 계산**하십시오. 같은 값을 여러 곳에서 관리하면 불일치가 발생합니다.
- 필요 시 `useMemo`로 **계산 비용만 최적화**합니다.

```jsx
function List({ items, filter }) {
  const visible = useMemo(
    () => items.filter((x) => x.includes(filter)),
    [items, filter]
  );
  return (
    <ul>
      {visible.map((v) => (
        <li key={v}>{v}</li>
      ))}
    </ul>
  );
}
```

---

## Prop Drilling 완화

- **문제**: 깊은 하위 트리까지 동일한 props를 계속 전달해야 할 때 유지보수가 어려워집니다.
- **해결**
  - **Context API**: 전역/광역 상태를 공급
  - 상태 관리 라이브러리(Zustand, Redux 등) 또는 서버 상태(TanStack Query) 병행

```jsx
const ThemeContext = createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Layout />
    </ThemeContext.Provider>
  );
}

function Button() {
  const theme = useContext(ThemeContext);
  return <button data-theme={theme}>OK</button>;
}
```

---

## Controlled vs Uncontrolled

- **Controlled**: 입력값을 **state로 소유**하고 `value`/`onChange`로 관리 → 폼 검증/동기화 용이
- **Uncontrolled**: DOM이 값을 소유(`defaultValue`, `ref`로 접근) → 간단한 폼에 유리

```jsx
// Controlled
<input value={value} onChange={(e) => setValue(e.target.value)} />

// Uncontrolled
<input defaultValue="hello" ref={inputRef} />
```

---

## 모범 사례 체크리스트

- [ ] 변경이 필요한 값인가? → **state**로 관리
- [ ] 외부에서 주입되는 값인가? → **props**
- [ ] 동일한 정보를 **중복 저장**하고 있지 않은가
- [ ] 변경은 값을 **소유한 곳**에서만 수행되는가
- [ ] 깊은 트리 전달 문제는 **Context/상태 관리**로 완화했는가
- [ ] 무거운 계산은 **메모화**했는가

---

## 요약

- **props**: 외부에서 주입되는 **읽기 전용 입력값**. 단방향 데이터 흐름을 보장합니다.
- **state**: 컴포넌트 내부의 **가변 데이터**. 변경 시 렌더링을 유발합니다.
- **자식이 변경해야 하는 경우**: 상태를 **부모로 끌어올리고 콜백**으로 의도를 전달하십시오.
