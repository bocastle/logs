# 널 오브젝트 패턴(Null Object Pattern) 정리

## 핵심 요약

- 널 오브젝트 패턴은 **객체가 없을 때 `null`을 전달하지 않고**, **아무 일도 하지 않는 객체(Null Object)** 를 전달하는 기법입니다.
- 반복되는 **널 체크 코드**를 줄여 코드가 단순해지고, 호출 측의 분기 처리가 감소합니다.
- 반면, 예외를 던져야 할 상황을 Null Object가 “조용히” 삼켜 **문제 탐지가 어려워질 수** 있습니다.

## 널 오브젝트 패턴이란?

널 오브젝트 패턴(Null Object Pattern)이란 객체가 존재하지 않을 때 `null`을 전달하는 것이 아니라, **아무 일도 하지 않는 객체**를 전달하는 기법입니다.

예를 들어 개발하다 보면 아래처럼 널 체크 코드를 작성할 때가 많습니다.

```java
public void doSomething(MyObject obj) {
  if (obj == null) {
    throw new Exception();
  }

  obj.doMethod();
}
```

이런 유형의 코드가 여러 곳에서 계속 반복해서 등장하면 코드가 복잡해질 수 있습니다.  
널 오브젝트 패턴은 널 값을 **아무런 행위도 하지 않는 객체로 다뤄** 널 체크 코드를 간소화합니다.

## 예시 코드

아래는 `MyObject`가 없을 때 `MyNullObject`를 사용해 호출 측 분기 없이 동작하도록 만든 예시입니다.

```java
class MyNullObject implements MyObject {
  @Override
  public void doMethod() {
    // 아무것도 하지 않음
  }
}

class MyRealObject implements MyObject {
  @Override
  public void doMethod() {
    System.out.println("무엇인가 수행합니다.");
  }
}

public void doSomething(MyObject obj) {
  obj.doMethod();
}
```

## 응용 사례

- 값이 `null`인 경우에만 사용하는 것이 아니라, **특별한 케이스**에도 응용할 수 있습니다.
- 예를 들어 스택(Stack) 자료구조를 만들 때 용량이 0인 경우, `ZeroCapacityStack` 같은 형태로 만들 수 있습니다.

## 장점

- 반복적인 널 체크 코드를 줄여 **코드를 간소화**할 수 있습니다.
- 협력을 재사용하는 데 용이하다는 장점이 있습니다.

## 단점

- 오히려 예외를 탐지하기 어려운 상황을 만들 수 있습니다.  
  (예: 원래는 예외로 드러나야 할 문제가 Null Object 때문에 조용히 지나갈 수 있음)

## 주의사항 / 실무 팁

- Null Object는 “아무것도 하지 않음”이 의도일 때만 쓰는 것이 안전합니다.  
  **실패를 드러내야 하는 케이스**라면 예외/에러 처리 방식을 유지하는 편이 좋습니다.
- Null Object가 반환/전달되는 상황을 로깅하거나, 도메인 규칙상 허용되는 케이스인지 명확히 문서화하면 디버깅에 도움이 됩니다.
- “널 체크 제거” 자체가 목적이 아니라, **호출 측 분기 제거로 인한 책임 이동**이 설계적으로 타당한지 먼저 점검하는 것이 좋습니다.
