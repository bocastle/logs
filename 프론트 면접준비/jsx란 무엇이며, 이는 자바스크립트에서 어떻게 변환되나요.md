# jsx란 무엇이며, 이는 자바스크립트에서 어떻게 변환되나요? (JavaScript XML)

## 정의

JSX는 **JavaScript XML**의 약자로, **React에서 UI를 선언적으로 표현하기 위해 사용하는 확장 문법**입니다.  
HTML과 유사한 문법을 사용하여 JavaScript 코드 안에서 UI 구조를 직관적으로 표현할 수 있습니다.

---

## 변환 과정

JSX는 브라우저에서 직접 실행되지 않습니다.  
**Babel** 같은 트랜스파일러를 통해 일반적인 JavaScript 코드로 변환된 후 실행됩니다.

### JSX 코드

```jsx
const element = <h1 className="greeting">Hello, JSX!</h1>;
```

### 변환된 JavaScript 코드

```js
const element = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);
```

👉 `React.createElement()`는 **가상 DOM(Virtual DOM) 요소를 생성**하는 API로,  
React가 변경 사항을 감지하고 실제 DOM에 효율적으로 반영할 수 있도록 합니다.

---

## React 외에서도 JSX 사용 가능?

JSX는 React 전용 문법이 아닙니다.  
단순히 JavaScript의 문법 확장일 뿐이므로, **트랜스파일링 환경**만 갖추면 다른 라이브러리나 프레임워크에서도 사용할 수 있습니다.

예를 들어, **Preact** 같은 React 대체 라이브러리나 JSX를 지원하는 렌더러에서도 활용 가능합니다.

---

## 하나의 부모 요소로 감싸야 하는 이유

JSX는 내부적으로 **JavaScript 객체**로 변환됩니다.  
JavaScript 함수는 **여러 개의 객체를 한 번에 반환할 수 없기 때문에** JSX도 반드시 하나의 부모 요소로 감싸야 합니다.

### 잘못된 예시 (에러 발생)

```jsx
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

### 올바른 예시 (부모 요소로 감싸기)

```jsx
return (
  <div>
    <h1>Hello</h1>
    <p>World</p>
  </div>
);
```

### Fragment 사용 (불필요한 DOM 요소 방지)

```jsx
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

---

## 정리

- JSX는 **UI를 직관적으로 표현하는 문법 확장**
- **Babel → JavaScript 코드 → React.createElement** 형태로 변환
- React 전용이 아니며, 다른 라이브러리에서도 사용 가능
- JSX 반환 시 반드시 **하나의 부모 요소**(또는 Fragment)로 감싸야 함

---
