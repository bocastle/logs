# React의 리렌더링 과정에 대해 설명

React의 리렌더링 과정은 크게 **Trigger**, **Render**, **Commit**이라는 세 단계로 나눌 수 있습니다.

---

## 🧩 Trigger 단계

컴포넌트의 **state나 props가 변경**되면서 시작됩니다.  
사용자의 입력, 네트워크 응답 등의 이벤트에 의해 상태가 변경되면 React는 해당 컴포넌트를 **다시 렌더링해야 한다고 판단**합니다.  
이때 React는 내부적으로 **업데이트 큐(update queue)** 에 해당 변경 사항을 등록합니다.

---

## 🎨 Render 단계

변경된 상태를 바탕으로 **새로운 Virtual DOM 트리**를 생성합니다.  
그 후, **이전 Virtual DOM과 새 Virtual DOM을 비교(diffing)** 하여 어떤 부분이 바뀌었는지를 분석합니다.

> ⚠️ 이 시점에서는 실제 DOM에는 아무런 변경도 일어나지 않습니다.  
> 오직 메모리 상의 Virtual DOM 비교만 수행됩니다.

---

## ⚡ Commit 단계

이전 단계에서 분석된 변경 사항을 실제 **DOM에 반영**합니다.  
React는 **변경에 필요한 최소한의 작업만 적용**하여 DOM을 업데이트합니다.  
변경이 발생하지 않은 요소는 수정하지 않고 그대로 둡니다.  
이 단계에서 사용자에게 **화면의 변화가 실제로 나타나게** 됩니다.

---

## 🤔 setState()가 호출될 때마다 매번 리렌더링이 발생하나요?

**아니요.**  
React의 **auto batching 기능**으로 인해, 여러 개의 상태 변경이 **자동으로 하나의 batch로 묶여서 처리**됩니다.

---

## 💡 예제 코드

```js
import { useState } from "react";

function App() {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);

  const handleClick = () => {
    setA(a + 1);
    setB(b + 1); // 이 둘은 하나로 합쳐져서 리렌더 1번
  };

  return <button onClick={handleClick}>Click</button>;
}
```

## 요약

| 단계    | 설명                                          |
| ------- | --------------------------------------------- |
| Trigger | state, props 변경으로 인해 업데이트 요청 발생 |
| Render  | 새로운 Virtual DOM 생성 및 diffing 수행       |
| Commit  | 실제 DOM에 최소 변경 반영                     |
