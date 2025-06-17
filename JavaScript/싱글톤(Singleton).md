# 싱글톤 (Singleton)

---

## 개념

- 애플리케이션 내에서 단 하나의 인스턴스만 생성되어야 할 때 사용
- 전역 상태 관리, 설정 객체 등에 활용

## 특징

- 인스턴스가 하나만 존재
- 어디서든 동일한 인스턴스에 접근 가능

## 예제 (JavaScript)

```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    this.timestamp = Date.now();
    Singleton.instance = this;
  }

  getTimestamp() {
    return this.timestamp;
  }
}

// 사용 예
const instance1 = new Singleton();
const instance2 = new Singleton();

console.log(instance1 === instance2); // true
console.log(instance1.getTimestamp());
console.log(instance2.getTimestamp());
```
