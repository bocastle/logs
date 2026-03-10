# 리액트에서 변수 관리 시 `useRef`와 `let`의 차이

## 핵심 요약

- 컴포넌트 **내부**의 `let` 변수는 **리렌더링마다 초기화**되어 이전 값을 잃습니다.
- `useRef()`로 만든 값은 **리렌더링되어도 유지**되며, 값이 바뀌어도 **리렌더링을 유발하지 않습니다**.
- 컴포넌트 **외부**의 `let`은 리렌더링 영향은 받지 않지만, **공유/전역처럼 동작**해 권장되지 않습니다.

## `useRef()` vs `let` (컴포넌트 내부 기준)

### 리렌더링 시 동작 방식

- `let`으로 선언한 변수는 컴포넌트가 **리렌더링될 때마다 초기화**되어서 이전 값을 잃어버립니다.
- `useRef()`로 만든 변수는 컴포넌트가 **리렌더링되어도 값이 유지**됩니다.

### 리렌더링 유발 여부

- `useRef()`는 `useState()`와 다르게 **ref 값이 변경되어도 컴포넌트 리렌더링이 유발되지 않습니다**.

## 사용 사례

### DOM 요소 접근

`useRef()`는 주로 DOM 요소에 접근할 때 사용합니다. 예를 들어 `input`에 focus를 주거나 스크롤 위치를 조정하는 경우에 `ref`를 활용합니다.

```tsx
import { useEffect, useRef } from "react";

export default function Example() {
  const inputRef = useRef<HTMLInputElement | null>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder="자동 포커스" />;
}
```
