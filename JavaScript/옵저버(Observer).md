# 옵저버 (Observer)

---

## 개념

- 객체의 상태 변화를 관찰하는 옵저버(구독자)들이 있고, 주체(발행자)가 상태 변화를 알리면 옵저버들이 자동으로 업데이트되는 디자인 패턴
- 발행-구독(pub-sub) 패턴과 유사

## 왜 사용하는가?

- 객체 간 느슨한 결합을 유지하면서 상태 변화에 반응하도록 함
- 여러 객체가 한 객체의 상태 변화를 동기화할 때 효과적

## 특징

- 주체(Subject)는 상태 변화 시 옵저버들에게 알림
- 옵저버들은 주체에 등록/해제 가능
- 이벤트 기반 프로그래밍에 자주 사용됨

## JavaScript 예제

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  unsubscribe(observer) {
    this.observers = this.observers.filter((obs) => obs !== observer);
  }

  notify(data) {
    this.observers.forEach((observer) => observer.update(data));
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }

  update(data) {
    console.log(`${this.name} received data: ${data}`);
  }
}

// 사용 예
const subject = new Subject();

const observerA = new Observer("옵저버 A");
const observerB = new Observer("옵저버 B");

subject.subscribe(observerA);
subject.subscribe(observerB);

subject.notify("안녕하세요!");
// 옵저버 A received data: 안녕하세요!
// 옵저버 B received data: 안녕하세요!

subject.unsubscribe(observerA);

subject.notify("두번째 알림");
// 옵저버 B received data: 두번째 알림
```
