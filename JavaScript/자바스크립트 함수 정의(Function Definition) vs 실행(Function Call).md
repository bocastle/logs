
## 함수 정의 vs 함수 실행

### 1. 함수 정의 (Function Definition)

```js
const renderTable = (data) => {
  console.log("테이블 렌더링 중", data);
};
```

이 코드는 함수 자체를 **정의**만 해둔 것입니다.  
아직 아무 동작도 하지 않습니다.  
말 그대로 "필요할 때 이렇게 동작해라"는 지침만 적어놓은 상태입니다.

---

### 2. 함수 실행 (Function Call)

```js
renderTable(["사과", "배", "포도"]);
```

이제 정의해 둔 함수를 실제로 **실행**한 것입니다.  
이 시점에서 console에 로그가 출력되고, 함수 안의 동작이 수행됩니다.

---

### 정리

| 코드                             | 의미               |
|----------------------------------|--------------------|
| `const renderTable = (...) => {}` | 함수 정의          |
| `renderTable(...)`               | 함수 실행          |

---

## 클래스나 객체 내부에서 `this.renderTable` 이란?

React나 일반 클래스 문법에서 다음과 같은 코드를 자주 보게 됩니다:

```js
this.renderTable(data);
```

여기서 `this`는 **현재 클래스 인스턴스(또는 객체 자신)**를 가리킵니다.  
즉, `renderTable`이라는 함수가 **그 객체 안에 정의돼 있어야** 사용할 수 있습니다.

---

### 예시

```js
class MyComponent {
  renderTable = (data) => {
    console.log("테이블 렌더링 중", data);
  };

  render() {
    this.renderTable(["🍕", "🍔", "🍟"]);
  }
}
```

위 예시에서 `this.renderTable()`은 클래스 내부의 `renderTable` 메서드를 실행하는 방식입니다.

---

## renderTable: 실제 테이블 렌더링 예제

HTML에서 간단한 테이블을 그리는 예제입니다.  
자바스크립트로 데이터를 받아서 동적으로 테이블을 생성합니다.

```js
const renderTable = (data) => {
  let html = '<table border="1">';
  html += '<tr><th>이름</th><th>나이</th></tr>';

  data.forEach((item) => {
    html += `<tr><td>${item.name}</td><td>${item.age}</td></tr>`;
  });

  html += '</table>';
  document.body.innerHTML = html;
};

const sampleData = [
  { name: "홍길동", age: 25 },
  { name: "김영희", age: 30 },
  { name: "이철수", age: 28 },
];

renderTable(sampleData);
```

이 코드를 실행하면 브라우저에서 표 형식으로 데이터가 출력됩니다.

---

## 일반 함수 vs 화살표 함수에서의 this

클래스 내부에서 함수를 정의할 때 `this`가 의도한 대로 작동하려면,  
보통 **화살표 함수**를 사용하는 것이 안전합니다.

### 일반 함수로 정의할 경우

```js
renderTable(data) {
  console.log(this); // 예상과 다른 객체를 가리킬 수 있음
}
```

위처럼 일반 함수로 정의하면 `this`가 undefined이거나,  
원하지 않는 객체를 참조하는 문제가 발생할 수 있습니다.

### 화살표 함수로 정의할 경우

```js
renderTable = (data) => {
  console.log(this); // 클래스 인스턴스를 안정적으로 가리킴
}
```

화살표 함수는 상위 컨텍스트의 `this`를 유지하기 때문에  
React 클래스 컴포넌트 등에서 자주 사용됩니다.

---

## 요약

- `const 함수이름 = () => {}` → 함수 정의  
- `함수이름()` → 함수 실행  
- `this.함수이름()` → 클래스나 객체 안에서 해당 함수 실행  
- 화살표 함수(`=>`)는 `this`를 안정적으로 유지할 수 있어 유리함

---

기초적인 함수 호출 개념이지만,  
클래스 문법과 결합되면 더 복잡하게 느껴질 수 있습니다.  
이제는 구조와 흐름을 정확히 이해하고 활용할 수 있을 거예요.