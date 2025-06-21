# 전략 (Strategy)

---

## 개념

- 알고리즘 군을 정의하고 각각을 캡슐화하여 교환 가능하게 만드는 패턴
- 실행 시점에 알고리즘을 선택할 수 있도록 유연성을 제공

## 왜 사용하는가?

- 알고리즘을 클라이언트 코드에서 분리해 독립적으로 관리 가능
- 알고리즘 변경 시 클라이언트 코드 수정 없이 새로운 전략으로 교체 가능
- 조건문 없이 다양한 알고리즘을 쉽게 적용

## 특징

- 공통 인터페이스를 가진 여러 전략 클래스가 존재
- 컨텍스트(Context) 클래스가 전략 객체를 받아 실행

## JavaScript 예제

```javascript
// 전략 인터페이스 역할 (명시적 인터페이스는 없지만 역할 분리)
class StrategyAdd {
  execute(a, b) {
    return a + b;
  }
}

class StrategyMultiply {
  execute(a, b) {
    return a * b;
  }
}

// 컨텍스트 클래스
class Context {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  executeStrategy(a, b) {
    return this.strategy.execute(a, b);
  }
}

// 사용 예
const addStrategy = new StrategyAdd();
const multiplyStrategy = new StrategyMultiply();

const context = new Context(addStrategy);
console.log(context.executeStrategy(5, 3)); // 8

context.setStrategy(multiplyStrategy);
console.log(context.executeStrategy(5, 3)); // 15
```
