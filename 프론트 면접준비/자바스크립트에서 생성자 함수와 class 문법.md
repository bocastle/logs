# 자바스크립트에서 생성자 함수와 class 문법

## 생성자 함수란?

자바스크립트에서 **생성자 함수(Constructor Function)**는 객체를 생성하는 하나의 방법입니다.  
일반 함수처럼 `function` 키워드로 정의하지만, `new` 키워드와 함께 호출할 경우 새로운 객체가 만들어집니다.

생성자 함수 내부의 `this`는 새롭게 생성된 객체를 가리키며, 여기에 속성을 추가하면 해당 객체에 저장됩니다.

### 예시

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function () {
  console.log(`안녕하세요, 저는 ${this.name}입니다.`);
};

const person1 = new Person("Alice", 25);
person1.greet(); // 안녕하세요, 저는 Alice입니다.
```

## class 문법은 왜 도입되었나? 🤔

생성자 함수는 객체를 생성할 수 있지만 몇 가지 **단점**이 있습니다.

- **명확한 클래스 개념 부족** → 상속을 구현하려면 프로토타입 체인을 사용해야 해서 가독성이 좋지 않음
- **객체지향 언어와 차이** → Java, C++ 같은 다른 OOP 언어에 익숙한 개발자들에게는 이해하기 어려움
- **혼동 유발** → `new` 키워드 없이 일반 함수처럼 호출될 수도 있어 문제가 발생할 수 있음

이러한 한계를 극복하기 위해 **ES6에서 class 문법이 도입**되었습니다.

---

## class 문법 예시

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`안녕하세요, 저는 ${this.name}입니다.`);
  }
}

const person2 = new Person("Bob", 30);
person2.greet(); // 안녕하세요, 저는 Bob입니다.
```

## class 문법의 장점

- 생성자와 메서드를 명확하게 정의할 수 있어 **코드 가독성 향상**
- 다른 객체지향 언어와 유사한 문법으로 **이해하기 쉬움**
- `extends`, `super`를 활용해 **상속을 간결하게 구현 가능**
- `static`, `getter/setter` 등 **객체지향 관련 기능 제공**
- 생성자 함수와 달리 **일반 함수처럼 호출할 수 없도록 제한** → 혼동 방지

---

## 요약

- **생성자 함수**: 객체를 생성할 수 있는 방법이지만, 문법이 직관적이지 않고 유지보수가 어려움
- **class 문법**: 생성자 함수의 단점을 개선하여, 객체지향 프로그래밍 스타일을 명확하고 간결하게 지원하는 문법
