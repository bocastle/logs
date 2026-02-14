# this 바인딩 정리 (JavaScript)

자바스크립트에서 `this`는 **“어디서 정의됐는지”가 아니라 “어떻게 호출됐는지”**에 따라 결정됩니다.  
(단, **화살표 함수는 예외**로 “어디서 정의됐는지(렉시컬 스코프)”를 따릅니다.)

---

## 1) 전역 호출 (일반 함수 호출)

```js
function f() {
  console.log(this);
}
f();
```

- **브라우저(느슨한 모드)**: `this === window`
- **strict mode**: `this === undefined`
- **ESM(모듈) 최상위 스코프**: 최상위 `this`는 `undefined`인 경우가 일반적

> 실무에서 가장 흔한 함정: “전역 호출”은 환경/모드에 따라 `this`가 달라질 수 있습니다.

---

## 2) 메서드 호출 (obj.method())

```js
const obj = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
};

obj.greet(); // "Alice"
```

- `this`는 **점(.) 앞의 객체**(`obj`)로 바인딩됩니다.
- “메서드로 호출”이 핵심입니다.

### 주의: 메서드를 변수에 담으면 전역 호출로 변합니다

```js
const greet = obj.greet;
greet(); // (strict면 undefined, 아니면 window) -> obj를 잃음
```

---

## 3) 생성자 호출 (new)

```js
function Person(name) {
  this.name = name;
}

const p = new Person("Alice");
console.log(p.name); // "Alice"
```

- `new`를 붙이면 `this`는 **새로 생성된 인스턴스**를 가리킵니다.
- 클래스(`class`)도 동일한 규칙을 따릅니다.

---

## 4) 명시적 바인딩 (call / apply / bind)

```js
function greet() {
  console.log(this.name);
}

const user = { name: "Alice" };

greet.call(user); // "Alice"
greet.apply(user); // "Alice"
const bound = greet.bind(user);
bound(); // "Alice"
```

- `call/apply`는 즉시 실행하면서 `this`를 지정
- `bind`는 **새 함수를 반환**하고, 그 함수의 `this`를 고정

### bind vs new 우선순위

일반적으로 `bind`된 함수라도 `new`로 호출하면 `new`가 우선하는 케이스가 있습니다(표준 동작).  
즉, `new (greet.bind(user))()`처럼 쓰면 `this`는 새 인스턴스가 됩니다.

---

## 5) 화살표 함수 (lexical this)

화살표 함수는 **자기만의 this를 만들지 않고**, **바깥(상위 스코프)의 this를 그대로 캡처**합니다.

```js
const obj = {
  name: "Alice",
  greet: () => console.log(this.name),
};

obj.greet(); // 보통 undefined (전역 this에 name 없음)
```

- 위 예시는 `greet`가 **obj의 메서드처럼 보이지만**,
  화살표 함수는 **obj로 this가 바인딩되지 않습니다.**
- 화살표 함수는 콜백에서 “this를 잃지 않게” 할 때 유용합니다.

```js
function Counter() {
  this.count = 0;
  setInterval(() => {
    this.count++;
    console.log(this.count);
  }, 1000);
}

new Counter(); // 화살표가 바깥 this(인스턴스)를 유지
```

---

## 6) DOM 이벤트 핸들러

### (1) addEventListener + 일반 함수

```js
button.addEventListener("click", function () {
  console.log(this); // button (리스너가 붙은 요소)
  console.log(event.currentTarget); // 보통 this와 동일
  console.log(event.target); // 실제 클릭된 가장 안쪽 요소
});
```

- `this`는 기본적으로 **리스너가 등록된 요소**를 가리키는 경우가 많습니다.
- 다만 실제로는 `event.currentTarget`을 쓰는 편이 더 명확합니다.

### (2) addEventListener + 화살표 함수

```js
button.addEventListener("click", () => {
  console.log(this); // 상위 스코프의 this (DOM 요소 아님)
});
```

- 화살표 함수는 lexical this이므로, DOM 요소로 바인딩되지 않습니다.

### (3) 인라인 핸들러(onclick="...")

인라인 핸들러에서는 `this`가 요소를 가리키는 동작이 흔하지만(브라우저),
권장 방식이 아니므로 실무에서는 `addEventListener` + `event.currentTarget`을 선호합니다.

---

## 7) (추가로 자주 묻는) 클래스 메서드와 this 분실

React/일반 클래스에서 메서드를 콜백으로 넘기면 `this`를 잃는 문제가 흔합니다.

```js
class A {
  constructor() {
    this.name = "Alice";
    this.say = this.say.bind(this); // 해결 1) bind
  }
  say() {
    console.log(this.name);
  }
}
```

또는 클래스 필드 + 화살표로 해결하기도 합니다.

```js
class A {
  name = "Alice";
  say = () => {
    console.log(this.name); // 해결 2) lexical this
  };
}
```

---

## this 바인딩 규칙 요약

1. **new로 호출** → 새 인스턴스
2. **call/apply/bind** → 지정한 객체(단, new가 개입하면 예외적 우선순위 존재)
3. **obj.method()** → 점(.) 앞의 객체
4. **그 외 일반 호출** → strict면 `undefined`, 아니면 전역 객체
5. **화살표 함수** → 상위 스코프의 `this`를 그대로 사용(lexical)

---

## 실무 팁

- DOM/이벤트에서는 `this`보다 **`event.currentTarget`**이 더 안전하고 명확합니다.
- 콜백으로 메서드를 넘길 땐 `this`가 깨지기 쉬우니 **bind** 또는 **화살표(lexical this)**를 고려합니다.
- strict mode/ESM 환경을 염두에 두고 전역 `this`에 의존하지 않는 습관이 좋습니다.
