# 널 오브젝트 패턴 (Null Object Pattern)

## 정의

널 오브젝트 패턴은 객체가 존재하지 않을 때, **`null`을 직접 전달하지 않고 아무 동작도 하지 않는 객체를 전달하는 패턴**입니다.  
이 방식은 반복적인 `null` 체크 코드를 줄여 코드의 단순성과 안정성을 높이는 데 유용합니다.

---

## 문제 상황

일반적으로는 다음과 같이 `null` 체크가 필요한 경우가 많습니다.

```java
public void doSomething(MyObject obj) {
    if (obj == null) {
        throw new Exception();
    }
    obj.doMethod();
}
```

- 여러 메서드에서 null 체크가 반복됨
- 코드 복잡도 증가 및 가독성 저하

## 널 오브젝트 패턴 적용

### 인터페이스

```java
interface MyObject {
    void doMethod();
}
```

### 실제 객체

```java
class MyRealObject implements MyObject {
    @Override
    public void doMethod() {
        System.out.println("무엇인가 수행합니다.");
    }
}

```

### 널 객체

```java
class MyNullObject implements MyObject {
    @Override
    public void doMethod() {
        // 아무것도 하지 않음
    }
}
```

### 클라이언트 코드

```java
public void doSomething(MyObject obj) {
    obj.doMethod(); // null 체크 필요 없음
}

```

## 응용 사례

널 오브젝트 패턴은 단순히 `null`을 대체하는 것뿐 아니라, **특수한 케이스 처리**에도 활용할 수 있습니다.

- **스택 구현**: 용량이 0인 경우 `ZeroCapacityStack` 객체 활용
- **로깅 시스템**: 로거가 없는 경우 `NullLogger` 객체 활용
- **옵션 처리**: 특정 기능이 비활성화된 경우 `NullOption` 객체 활용

---

## 장점

- 반복되는 **`null` 체크 제거**
- 협력 객체 간 **일관된 인터페이스 제공**
- 코드의 **가독성**과 **유지보수성** 향상

---

## 단점

- 예외 발생을 숨길 수 있어 **문제를 조기에 탐지하기 어려움**
- 잘못 적용 시 **버그 추적 난이도 상승**

---
