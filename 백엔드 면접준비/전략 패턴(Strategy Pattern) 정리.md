# 전략 패턴(Strategy Pattern) 정리

전략 패턴은 **행위(알고리즘)를 객체로 캡슐화**해서, **런타임에 교체 가능**하도록 만드는 디자인 패턴입니다.  
핵심은 “조건문으로 분기하며 행동을 바꾸는 것”을 “전략 객체를 바꿔 끼우는 것”으로 치환하는 것입니다.

---

## 1) 언제 쓰나요?

- **같은 목적(행위)** 을 달성하는 방법이 여러 개인데, 상황에 따라 **알고리즘을 바꿔야** 할 때
  - 결제 수단(카드/계좌/포인트)
  - 할인 정책(정액/정률/회원등급)
  - 배송비 계산(지역/무게/프로모션)
  - 정렬 기준(이름/가격/최신순)
- 기존 코드에 `if/else`, `switch`가 계속 늘어나고, 새로운 규칙 추가 시 **기존 코드 수정이 반복**될 때

---

## 2) 구성 요소

- **Context(문맥)**: 전략을 사용(호출)하는 쪽. 전략을 “선택”하거나 “주입”받아 사용합니다.
- **Strategy(전략 인터페이스/추상화)**: 알고리즘(행위)의 공통 계약(메서드)을 정의합니다.
- **ConcreteStrategy(구현 전략)**: 실제 알고리즘 구현체입니다.

---

## 3) 장점

- **OCP(개방-폐쇄 원칙)** 에 유리: 새 전략을 추가해도 기존 Context 코드를 덜/안 건드립니다.
- 조건 분기 감소 → **가독성/테스트 용이성** 향상
- 전략 단위로 **독립적인 테스트**가 가능

---

## 4) 단점/주의사항

- 클래스(전략) 수가 늘 수 있습니다.
- 전략 선택 로직(어떤 전략을 쓸지)이 필요합니다.
  - 선택 로직이 다시 거대한 `if/else`가 되지 않도록 주의(팩토리/레지스트리/맵 기반 선택 등 활용)

---

## 5) 예시 (질문에서 제시한 형태 개선 포함)

### 5-1. 전략 인터페이스

```java
public interface MoveStrategy {
    boolean isMovable(int input);
}
```

### 5-2. 구현 전략들

```java
public class EvenNumberMoveStrategy implements MoveStrategy {
    @Override
    public boolean isMovable(int input) {
        return input % 2 == 0;
    }
}

public class OddNumberMoveStrategy implements MoveStrategy {
    @Override
    public boolean isMovable(int input) {
        return input % 2 != 0;
    }
}
```

### 5-3. Context (Car)

> 팁: `position`이 불변이라면, `move`가 새 객체를 반환(불변 객체)하는 설계도 가능하고,  
> 가변 객체로 두고 내부 상태를 변경하는 설계도 가능합니다(도메인/팀 스타일에 맞게).

```java
public class Car {
    private final MoveStrategy strategy;
    private final int position;

    public Car(MoveStrategy strategy, int position) {
        this.strategy = strategy;
        this.position = position;
    }

    public Car move(int input) {
        if (strategy.isMovable(input)) {
            return new Car(strategy, position + 1);
        }
        return this;
    }

    public int getPosition() {
        return position;
    }
}
```

---

## 6) 전략 선택(주입) 방법

### 6-1. 생성 시점에 주입(가장 단순)

```java
Car car = new Car(new EvenNumberMoveStrategy(), 0);
car = car.move(2);
```

### 6-2. 런타임에 교체(동적으로 바꾸기)

전략을 필드로 두고 바꿔 끼우고 싶다면(가변 설계):

```java
public class Car {
    private MoveStrategy strategy;
    private int position;

    public void changeStrategy(MoveStrategy strategy) {
        this.strategy = strategy;
    }
}
```

다만, 도메인 규칙상 “차의 이동 규칙이 도중에 바뀌는 게 맞는가?”를 먼저 검토하는 것이 좋습니다.

### 6-3. (스프링) 전략 레지스트리 패턴으로 선택 로직 정리

전략이 많아질수록 `if/else` 대신 “키 → 전략” 매핑을 권장합니다.

```java
public interface DiscountStrategy {
    String key();              // 예: "FIXED", "RATE"
    long discount(long price); // 할인 계산
}
```

```java
@Component
public class FixedDiscountStrategy implements DiscountStrategy {
    public String key() { return "FIXED"; }
    public long discount(long price) { return Math.max(0, price - 1000); }
}
```

```java
@Component
public class DiscountStrategyRegistry {
    private final Map<String, DiscountStrategy> strategies;

    public DiscountStrategyRegistry(List<DiscountStrategy> list) {
        this.strategies = list.stream()
                .collect(Collectors.toMap(DiscountStrategy::key, s -> s));
    }

    public DiscountStrategy get(String key) {
        DiscountStrategy strategy = strategies.get(key);
        if (strategy == null) throw new IllegalArgumentException("Unknown key: " + key);
        return strategy;
    }
}
```

```java
@Service
public class PriceService {
    private final DiscountStrategyRegistry registry;

    public PriceService(DiscountStrategyRegistry registry) {
        this.registry = registry;
    }

    public long finalPrice(long price, String discountType) {
        return registry.get(discountType).discount(price);
    }
}
```

---

## 7) 정리 한 줄

전략 패턴은 **변하는 알고리즘(행위)을 인터페이스 뒤로 숨기고**,  
**Context는 전략을 교체(주입)함으로써 동작을 바꾸는** 패턴입니다.
