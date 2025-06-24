# 📌 useRef는 언제 사용하나요? (React)

`useRef()`는 React의 훅 중 하나로, 컴포넌트 내에서 변경 가능한 값을 저장하고, **렌더링을 유발하지 않으면서도 값을 유지**할 수 있게 해줍니다. 주로 다음 두 가지 용도로 사용됩니다.

---

## 1️⃣ DOM 요소에 접근할 때

```jsx
import { useRef, useEffect } from "react";

function MyComponent() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus(); // 컴포넌트 마운트 시 input에 포커스를 설정
  }, []);

  return <input ref={inputRef} />;
}
```

### 📝 설명

- `useRef(null)`을 통해 input 요소를 참조할 준비를 합니다.
- `ref={inputRef}`로 실제 DOM에 연결합니다.
- `useEffect()` 안에서 `inputRef.current.focus()`로 포커스를 설정합니다.

---

## 2️⃣ 렌더링 없이 값 유지할 때

`useState()`와 달리 `useRef()`는 값이 변경되어도 컴포넌트를 리렌더링하지 않습니다.  
따라서 **타이머 ID**, **이전 값 추적**, **외부 라이브러리 인스턴스 보관** 등에 유용합니다.

```jsx
import { useRef } from "react";

function TimerComponent() {
  const timerRef = useRef(null);

  const startTimer = () => {
    timerRef.current = setInterval(() => {
      console.log("타이머 실행");
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerRef.current); // 타이머 정지
  };

  return (
    <>
      <button onClick={startTimer}>시작</button>
      <button onClick={stopTimer}>정지</button>
    </>
  );
}
```

### 📝 설명

- `timerRef.current`에 `setInterval`로 생성된 ID를 저장합니다.
- 이 값은 상태로 관리하지 않기 때문에 **리렌더링을 유발하지 않습니다.**

---

## ✅ 요약

| 사용 목적           | 설명                                                        |
| ------------------- | ----------------------------------------------------------- |
| DOM 접근            | 요소에 직접 접근하여 포커스 등 제어                         |
| 렌더링 없이 값 저장 | 값이 변해도 리렌더링되지 않음 (예: 타이머, 이전 값 추적 등) |

---

## 🔚 참고 사항

- `useRef`는 렌더링과 무관한 참조값을 저장하고 싶을 때 사용하는 도구입니다.
- **렌더링이 필요한 데이터는** `useState` 또는 `useReducer`를 사용해야 합니다.
