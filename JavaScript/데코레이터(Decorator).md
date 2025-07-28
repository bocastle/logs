# 데코레이터 (Decorator)

---

## 개념

- 기존 객체에 기능을 동적으로 추가하거나 확장하는 패턴
- 상속 대신에 객체를 감싸 기능을 추가하는 방식으로 유연하게 확장 가능

## 왜 사용하는가?

- 상속보다 더 유연하게 기능 확장 가능
- 런타임에 객체에 새로운 책임을 추가할 수 있음
- 기능을 조합하여 다양한 객체를 만들 때 유용

## 특징

- 원본 객체와 동일한 인터페이스를 가짐
- 데코레이터 객체가 원본 객체를 감싸고 추가 기능을 덧붙임

## JavaScript 예제

```javascript
// 기본 객체
class Coffee {
  cost() {
    return 5;
  }
}

// 데코레이터 클래스
class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost() + 2;
  }
}

class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost() + 1;
  }
}

// 사용 예
let myCoffee = new Coffee();
console.log(myCoffee.cost()); // 5

myCoffee = new MilkDecorator(myCoffee);
console.log(myCoffee.cost()); // 7

myCoffee = new SugarDecorator(myCoffee);
console.log(myCoffee.cost()); // 8
```
