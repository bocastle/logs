# SOLID 원칙 정리

> 목적: 변화에 **유연**하고 **유지보수성** 높은 객체지향 설계를 위해 의존성을 관리하는 다섯 가지 핵심 원칙입니다.

---

## S — 단일 책임 원칙 (Single Responsibility Principle)

**정의**: 클래스(모듈)는 **오직 하나의 변경 이유**만 가져야 합니다.  
**의미**: “책임”은 특정 사용자·요구사항의 변경 요청을 처리하는 **역할**을 뜻합니다.

**신호**

- 하나의 클래스가 **여러 액터의 요구**(예: 영업팀·회계팀)를 동시에 처리
- 서로 다른 변경 주기/이유의 코드가 **동거**

**대응**

- 관심사 분리(도메인 모델 ↔ 영속성 ↔ 프레젠테이션)
- 정책과 구현 분리, 기능 단위로 **모듈화**

```java
// 위반: 영수증 생성 + 파일 저장 + 이메일 전송까지 담당
class InvoiceService {
  void generate() {}
  void saveToFile() {}
  void sendEmail() {}
}

// 준수: 책임 분리
class InvoiceService { void generate() {} }
class InvoiceRepository { void save() {} }
class NotificationService { void sendEmail() {} }
```

---

## O — 개방/폐쇄 원칙 (Open–Closed Principle)

**정의**: **확장에는 열려** 있고, **변경에는 닫혀** 있어야 합니다.  
**핵심**: 새 기능은 **새 타입 추가**로 도입하고, 기존 상위 수준 모듈은 **수정 없이** 동작해야 합니다.

**전략**

- 다형성(인터페이스), **전략/템플릿/데코레이터** 패턴
- 조건 분기(`if/else` 열거) → **구성/등록 기반** 확장

```java
// 위반: 결제 수단이 늘 때마다 분기 추가
switch (method) { case CARD: ...; case CASH: ...; }
// 준수: 확장
interface Payment { void pay(int amt); }
class CardPay implements Payment { ... }
class CashPay implements Payment { ... }
```

> 실무 팁: **모든** 변화에 대비해 과도한 추상화를 미리 두지 마십시오.  
> 실제 변화가 발생하고 **반복될 때** 추상화를 도입해 OCP를 달성하십시오.

---

## L — 리스코프 치환 원칙 (Liskov Substitution Principle)

**정의**: **하위 타입은 상위 타입으로 대체 가능**해야 합니다(계약 준수).  
**의미**: 상위 타입에 대한 **사전/사후 조건, 불변식, 예외 규약**을 하위 타입이 **위배하지 않음**.

**전형적 위반**

- 상위 타입이 허용한 입력을 하위 타입이 **거부**
- 상위 타입이 보장한 결과/사이드이펙트를 하위 타입이 **약화**

```java
// 예시: Rectangle ↔ Square 문제(상위 계약 setWidth/Height가 깨짐)
```

**대응**

- 상속보다 **합성** 우선
- “is‑a” 관계를 **행동 계약** 기준으로 검증

---

## I — 인터페이스 분리 원칙 (Interface Segregation Principle)

**정의**: 클라이언트는 **사용하지 않는 메서드에 의존**하지 않아야 합니다.  
**의미**: 거대 인터페이스는 변경 파급을 키웁니다.

**대응**

- 역할별 **작은 인터페이스**로 분리
- 어댑터/파사드로 필요한 **뷰만 노출**

```java
interface Printer { void print(); void scan(); void fax(); }
// 분리
interface Printable { void print(); }
interface Scannable { void scan(); }
```

---

## D — 의존성 역전 원칙 (Dependency Inversion Principle)

**정의**: 상위 수준 모듈과 하위 수준 모듈 모두 **추상화**에 의존해야 합니다.  
**의미**: 구현(하위)이 추상(상위)을 따르도록 **의존 방향을 역전**.

**구현 방법**

- 인터페이스 도입 + **의존성 주입(DI)**, IoC 컨테이너
- 상위 모듈은 **정책**, 하위 모듈은 **세부 구현**

```java
interface MessageSender { void send(String msg); }
class EmailSender implements MessageSender { ... }
class OrderService {
  private final MessageSender sender;
  OrderService(MessageSender sender) { this.sender = sender; }
}
```

---

## 원칙 간 관계와 적용 순서 (실무 관점)

1. **SRP**로 책임을 **분리**하면 변경 파급을 줄입니다.
2. **LSP**를 지키는 타입 계층이 있어야 **OCP**가 의미를 가집니다.
3. **ISP**로 클라이언트별 계약을 **축소**하면 결합이 낮아집니다.
4. **DIP**로 상위 정책이 하위 구현에 **끌려가지 않게** 만듭니다.

---

## 과도한 추상화 지양: “지금 필요한 것만”

- 모든 변화를 **사전 예측**하려는 추상화는 **복잡도/비용**만 키웁니다.
- 실제 고객 요구로 **변화가 발생하고 반복**될 때, 그 **유형을 일반화**하여 추상화/확장점을 도입하십시오.
- 작은 리팩터링 주기, 테스트(단위/통합)로 안전망을 확보하십시오.

---

## 체크리스트

- [ ] 클래스/모듈의 **변경 이유**가 한 가지인가(SRP)
- [ ] 기능 확장은 **새 타입/구성**으로 가능하며 상위 코드는 수정이 적은가(OCP)
- [ ] 하위 타입이 상위 타입의 **행동 계약**을 위반하지 않는가(LSP)
- [ ] 인터페이스가 **역할 중심의 최소 집합**으로 분리되어 있는가(ISP)
- [ ] 상위 정책이 **추상화에 의존**하고 구현 세부에 끌려가지 않는가(DIP)
