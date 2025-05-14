# SOLID 원칙에 대해서 설명해 주세요

SOLID 원칙은 객체지향 설계 5원칙이라고도 불리며, 각 원칙의 앞 글자를 따서 만들어졌습니다. 객체지향설계의 핵심 중 하나는 의존성을 관리하는 것인데요. 의존성을 잘 관리하기 위해서는 SOLID 원칙을 준수해야 합니다.

---

## 1. 단일 책임 원칙 (Single Responsibility Principle)

클래스가 오직 하나의 목적이나 이유로만 변경되어야 한다는 것을 강조합니다.  
여기서 “책임”이란 단순히 메서드의 개수를 뜻하지 않고, 특정 사용자나 기능 요구사항에 따라 소프트웨어의 변경 요청을 처리하는 역할을 의미합니다.

즉, 클래스는 한 가지 변화의 이유만 가져야 하며, 이를 통해 변경이 발생했을 때 다른 기능에 영향을 덜 미치도록 설계됩니다.  
이렇게 하면 유지보수가 쉬워지고 코드가 더 이해하기 쉬워집니다.

---

## 2. 개방-폐쇄 원칙 (Open-Closed Principle)

확장에는 열려있고, 변경에는 닫혀 있어야 함을 강조합니다.

- 확장: 새로운 타입을 추가함으로써 새로운 기능을 추가
- 폐쇄: 확장이 일어날 때 상위 레벨의 모듈이 영향을 받지 않아야 함

이를 통해서 모듈의 행동을 쉽게 변경할 수 있습니다.  
모듈이란 크기와 상관없이 클래스, 패키지, 라이브러리와 같이 프로그램을 구성하는 임의의 요소를 의미합니다.

---

## 3. 리스코브 치환 원칙 (Liskov Substitution Principle)

서브 타입은 언제나 상위 타입으로 교체할 수 있어야 합니다.

즉, 서브 타입은 상위 타입이 약속한 규약을 지켜야 함을 강조합니다.  
이 원칙은 부모 쪽으로 업 캐스팅하는 것이 안전함을 보장하기 위해 존재합니다.

> LSP를 위반하는 대표적인 사례: Rectangle 예제

### 🔻 위반 예시 - Rectangle & Square

```javascript
// ✅ 부모 클래스: 직사각형
class Rectangle {
  constructor() {
    this.width = 0;
    this.height = 0;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

// ❌ 자식 클래스: 정사각형 (LSP 위반)
class Square extends Rectangle {
  // 정사각형은 가로와 세로가 항상 같아야 하므로
  // 하나만 바꿔도 다른 값까지 강제로 바꾼다.
  setWidth(width) {
    this.width = width;
    this.height = width;
  }

  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

// ✅ 상위 타입을 받아서 넓이를 테스트하는 함수
function testArea(rectangle) {
  rectangle.setWidth(5);
  rectangle.setHeight(10);

  // 예상 결과는 5 * 10 = 50
  console.log(`Expected area: 50, Actual area: ${rectangle.getArea()}`);
}

// 🔸 테스트
const rect = new Rectangle();
const square = new Square();

testArea(rect); // ✅ Expected area: 50, Actual area: 50
testArea(square); // ❌ Expected area: 50, Actual area: 100
```

상위 타입에 대해 기대되는 역할과 행동 규약이 있는데 이를 벗어나면 안 됩니다.  
만약 하위 타입이 이를 만족하지 않으면, OCP를 달성하기 어렵게 됩니다.

---

## 4. 인터페이스 분리 원칙 (Interface Segregation Principle)

클라이언트 입장에서 인터페이스를 분리해야 함을 강조합니다.

사용하지 않지만 의존성이 있으면, 해당 인터페이스가 변경될 경우 영향을 받게 됩니다.  
이는 독립적인 개발과 배포를 어렵게 합니다.

→ 사용하는 기능만 제공하도록 인터페이스를 분리해 **변경의 여파를 최소화**할 수 있습니다.

---

## 5. 의존성 역전 원칙 (Dependency Inversion Principle)

상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 되며,  
**모두 추상화에 의존해야 함**을 강조합니다.

SOLID 원칙들은 서로 연관되어 있으며,  
의존성 역전 원칙을 통해 하위 모듈은 개방-폐쇄 원칙을 준수하면서 새로운 타입이 추가 가능합니다.

---

## 💡 OCP는 미래 예측이 아닌, 변화에 대한 대응이다

중요한 것은 예상되는 변화가 **실제로 발생했을 때** 추상화를 적용하여, 같은 종류의 변화를 견딜 수 있도록 만드는 것입니다.

- 모든 경우에 대비하려 하지 마세요.
- 실제 변화가 일어난 후 **재사용 가능한 형태로 리팩터링**하세요.
