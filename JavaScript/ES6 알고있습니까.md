
# ES6 (ECMAScript 2015) 주요 기능

ECMAScript 2015, 즉 ES6는 자바스크립트의 최신 버전으로, 코드의 가독성 및 유지보수성을 향상시키기 위한 여러 기능을 추가했습니다. ES6의 주요 변경 사항을 아래와 같이 정리했습니다.

## 1. `let`과 `const`

- `let`과 `const`는 변수 선언을 위한 키워드로, **블록 스코프**를 가집니다.
- 반면, `var`는 **함수 스코프**를 가지며, 범위가 제한적입니다.
- `let`은 값을 재할당할 수 있으며, `const`는 상수로 선언하여 값을 변경할 수 없습니다.
- 두 키워드 모두 **호이스팅**이 발생하지만, **초기화 이전에 접근 시 `ReferenceError`**가 발생합니다.
  - 이는 **"일시적 사각지대(Temporal Dead Zone)"**라고 불립니다.

### 예시:

```javascript
function test() {
  console.log(a); // undefined
  var a = 10;
  
  console.log(b); // ReferenceError: Cannot access 'b' before initialization
  let b = 20;
}

test();
```

## 2. 화살표 함수 (Arrow Function)

- 화살표 함수는 기존의 함수 표현식을 더욱 간결하게 만들어줍니다.
- `function` 키워드를 사용하지 않고, `=>`를 사용하여 함수 선언을 할 수 있습니다.
- 또한, 화살표 함수는 `this` 바인딩이 호출 시점이 아니라 함수 선언 시점에 고정됩니다. 따라서 `this`에 대한 혼란이 줄어듭니다.

### 예시:

```javascript
const add = (a, b) => a + b;
console.log(add(2, 3)); // 5

const multiply = (a, b) => {
  return a * b;
};
console.log(multiply(3, 4)); // 12
```

## 3. 클래스 (Class)

- ES6에서는 객체지향 프로그래밍을 지원하는 **클래스** 문법이 추가되었습니다.
- 클래스를 사용하면 객체의 생성자 함수와 상속을 보다 쉽게 관리할 수 있습니다.

### 예시:

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hello, my name is ${this.name} and I am ${this.age} years old.`);
  }
}

const john = new Person('John', 30);
john.greet(); // Hello, my name is John and I am 30 years old.
```

## 4. 템플릿 리터럴 (Template Literal)

- 문자열을 처리할 때 더 간편하고 직관적인 방법을 제공합니다.
- `${}`를 사용하여 문자열 안에 변수나 표현식을 삽입할 수 있습니다.

### 예시:

```javascript
const name = 'John';
const age = 30;
const message = `Hello, my name is ${name} and I am ${age} years old.`;
console.log(message); // Hello, my name is John and I am 30 years old.
```

## 5. 구조 분해 할당 (Destructuring Assignment)

- 배열이나 객체에서 값을 쉽게 추출할 수 있도록 도와줍니다.
- 배열이나 객체의 속성을 변수에 쉽게 할당할 수 있습니다.

### 예시:

```javascript
// 배열 구조 분해 할당
const arr = [1, 2, 3];
const [a, b] = arr;
console.log(a, b); // 1 2

// 객체 구조 분해 할당
const person = { name: 'John', age: 30 };
const { name, age } = person;
console.log(name, age); // John 30
```

## 6. `Promise`

- 비동기 작업을 더 간편하게 처리할 수 있게 도와주는 **Promise**가 추가되었습니다.
- `resolve`와 `reject`로 처리 결과를 다루며, `.then()`, `.catch()` 메서드로 체이닝할 수 있습니다.

### 예시:

```javascript
const myPromise = new Promise((resolve, reject) => {
  const success = true;
  if (success) {
    resolve('Operation was successful!');
  } else {
    reject('Operation failed!');
  }
});

myPromise
  .then(result => console.log(result)) // Operation was successful!
  .catch(error => console.log(error));
```

## 7. `Spread` 연산자와 `Rest` 매개변수

- `Spread` 연산자는 배열이나 객체를 쉽게 복사하거나 병합할 때 유용합니다.
- `Rest` 매개변수는 함수의 매개변수를 배열로 받아, 가변 인자 리스트를 처리할 수 있게 해줍니다.

### 예시:

```javascript
// Spread 연산자
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];
console.log(arr2); // [1, 2, 3, 4, 5]

// Rest 매개변수
function sum(...args) {
  return args.reduce((acc, curr) => acc + curr, 0);
}
console.log(sum(1, 2, 3, 4)); // 10
```

## 8. `default` 파라미터

- 함수의 매개변수에 기본값을 설정할 수 있습니다.
- 매개변수가 전달되지 않으면 기본값이 자동으로 적용됩니다.

### 예시:

```javascript
function greet(name = 'Guest') {
  console.log(`Hello, ${name}!`);
}

greet('John'); // Hello, John!
greet(); // Hello, Guest!
```

## 9. `Map`과 `Set`

- ES6에서는 **Map**과 **Set**이라는 새로운 자료형이 추가되었습니다.
  - **Map**은 키-값 쌍을 저장하는 자료형입니다.
  - **Set**은 중복되지 않는 값들을 저장하는 자료형입니다.

### 예시:

```javascript
// Map
const map = new Map();
map.set('name', 'John');
map.set('age', 30);
console.log(map.get('name')); // John

// Set
const set = new Set([1, 2, 3, 3, 4]);
console.log(set); // Set { 1, 2, 3, 4 }
```

## 10. 모듈 (Module)

- ES6에서는 **모듈 시스템**이 추가되었습니다. 이를 통해 코드 분할 및 재사용이 용이해졌습니다.
- `import`와 `export` 키워드를 사용하여 다른 파일에서 코드나 변수들을 불러오고 내보낼 수 있습니다.

### 예시:

```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// main.js
import { add, subtract } from './math.js';
console.log(add(2, 3)); // 5
console.log(subtract(5, 3)); // 2
```

---
