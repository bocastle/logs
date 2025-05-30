# useEffect가 호출되는 시점에 대해 설명해 주세요.

React의 useEffect는 컴포넌트의 특정 시점에 자동으로 호출되는 훅으로, 크게 컴포넌트가 마운트, 업데이트, 언마운트되는 시점에 호출됩니다.

---

## 마운트

먼저, useEffect는 컴포넌트가 마운트될 때, 즉, 처음 렌더링되고 나서 호출됩니다. 이때 데이터 초기화나 외부 API 호출, 구독 설정 등의 작업을 실행할 수 있습니다. 이처럼 useEffect는 컴포넌트가 처음 마운트될 때 필요한 초기 작업을 수행할 수 있도록 해줍니다.

---

## 업데이트

또한, useEffect는 의존성 배열에 지정된 값이 변경될 때마다 다시 호출됩니다. 이때, useEffect의 return 값으로 지정된 클린업 함수가 이전 props 및 state와 함께 먼저 호출된 후, 본문의 실행 로직이 업데이트된 props 및 state와 함께 실행됩니다.
두 번째 인자로 주어지는 의존성 배열은 useEffect가 어떤 상태나 props의 변화에 반응할지를 결정합니다. 예를 들어 useEffect(() => {...}, [count])처럼 count 상태가 의존성 배열에 있을 경우, count 값이 변경될 때마다 useEffect가 호출됩니다. 이를 통해 특정 상태나 props가 변경될 때마다 필요한 동작을 수행하도록 할 수 있으며, 컴포넌트의 변화에 따라 동적으로 실행되는 로직을 설정할 수 있습니다.
단, 의존성 배열을 넘기지 않을 경우에는 매 렌더링마다 호출됩니다.

---

## 언마운트

마지막으로, 컴포넌트가 언마운트될 때 useEffect의 return 값으로 지정된 클린업 함수가 호출됩니다. 이 정리 함수를 이용하여 이벤트 리스너 제거, 타이머 해제, 구독 취소 등의 작업을 수행할 수 있습니다. 이를 통해 useEffect를 통해 발생한 부수효과를 정리하는 것입니다.

---

## 요약

요약하자면, useEffect는 컴포넌트가 처음 렌더링된 후, 의존성 배열의 값이 변경될 때, 그리고 컴포넌트가 언마운트될 때 호출됩니다.
