# 팩토리 (Factory)

---

## 개념

- 객체 생성 로직을 별도의 팩토리 클래스로 분리하는 디자인 패턴
- 생성 방식이 복잡하거나 여러 종류의 객체를 생성해야 할 때 사용

## 왜 사용하는가?

- 클라이언트 코드와 객체 생성 코드를 분리하여 응집도 향상
- 객체 생성 방식을 캡슐화해서 변경에 유연하게 대응 가능

## 특징

- 객체 생성 책임을 팩토리에게 위임
- 다양한 타입의 객체를 조건에 따라 생성할 수 있음

## JavaScript 예제

```javascript
class Dog {
  speak() {
    return "멍멍";
  }
}

class Cat {
  speak() {
    return "야옹";
  }
}

class AnimalFactory {
  static createAnimal(type) {
    switch (type) {
      case "dog":
        return new Dog();
      case "cat":
        return new Cat();
      default:
        throw new Error("알 수 없는 동물 타입입니다.");
    }
  }
}

// 사용 예
const dog = AnimalFactory.createAnimal("dog");
const cat = AnimalFactory.createAnimal("cat");

console.log(dog.speak()); // 멍멍
console.log(cat.speak()); // 야옹
```
