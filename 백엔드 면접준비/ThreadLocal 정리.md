# ThreadLocal 정리

## 핵심 요약(짧게)

- `ThreadLocal`은 Java에서 **스레드마다 독립적인 값을 저장**할 수 있게 해주는 클래스입니다.
- 공유 자원을 직접 공유하지 않아도 되므로, 동기화(`synchronized`) 없이 **스레드별로 안전하게 값 분리**가 가능합니다.
- 스레드풀 환경에서는 스레드가 재사용되므로, 사용 후 `remove()`로 값을 정리하지 않으면 **데이터 오염/메모리 누수** 문제가 생길 수 있습니다.

## ThreadLocal이란?

`ThreadLocal`은 Java에서 각 스레드마다 독립적인 변수를 저장할 수 있도록 도와주는 클래스입니다.

여러 스레드가 공유 자원을 사용하면 동시성 문제가 발생할 수 있는데, `ThreadLocal`을 사용하면 스레드별로 데이터를 분리할 수 있어 **동기화 없이 안전하게** 활용할 수 있습니다.

각 스레드는 자신만의 `ThreadLocalMap`을 가지고 있고, `ThreadLocal`을 키로 사용하여 값을 저장합니다.  
즉, 하나의 스레드에서 여러 개의 `ThreadLocal`을 사용할 수 있으며, `ThreadLocal`은 현재 스레드의 `ThreadLocalMap`을 제어하는 역할을 합니다.

## Spring 생태계에서의 활용 예

Spring 생태계에서는 `ThreadLocal`을 사용해 다음과 같은 기능을 제공합니다.

- 트랜잭션 동기화 관리: `TransactionSynchronizationManager`
- 사용자 인증 정보 관리: `SecurityContextHolder`
- 웹 요청 attribute 관리: `RequestContextHolder`

## 장점

- 각 스레드가 다른 스레드와 **격리된 값**을 가질 수 있습니다.
- 공유 자원이 없기 때문에 `synchronized` 같은 **동기화가 필요 없습니다.**

## 단점

- 원문에서는 별도의 단점이 직접적으로 나열되지는 않았지만, 아래 “주의사항”에 해당하는 리스크가 존재합니다.

## 주의사항 / 실무 팁

### 스레드풀 환경에서는 반드시 정리(`remove()`)하기

스레드풀을 사용하면 스레드가 재사용됩니다. 이때 `ThreadLocal`에 이전 값이 남아 있으면 재사용된 스레드가 올바르지 않은 데이터를 참조할 수 있습니다.  
이를 방지하려면 스레드가 끝나는 시점에 `remove()` 메서드를 호출해 값을 제거해야 합니다.

```java
ThreadLocal<String> ctx = new ThreadLocal<>();

try {
  ctx.set("value");
  // ... 사용
} finally {
  ctx.remove(); // 스레드 재사용/누수 방지
}
```

### 비동기 작업에서는 예상대로 동작하지 않을 수 있음

비동기 작업에서는 실행 스레드가 달라질 수 있어, 기존 스레드의 `ThreadLocal` 값이 전달되지 않을 수 있습니다.

예를 들어 `@Async`로 실행되는 작업은 새로운 스레드에서 수행되므로, 비동기 스레드는 기존 스레드에서 저장한 `ThreadLocal` 값을 참조할 수 없습니다.

- Spring 4.3 이상: `TaskDecorator`를 사용해 기존 스레드의 `ThreadLocal` 값을 비동기 스레드로 복사하는 방식으로 해결할 수 있습니다.

## ThreadLocal을 대체할 수 있는 방법

- 메서드 인자로 값을 전달하기
- `ConcurrentHashMap` 같은 thread-safe 자료구조 사용하기
- Spring에서 HTTP 요청 단위로 데이터 관리가 필요하면 `@RequestScope` 활용하기

## NamedThreadLocal이란?

`NamedThreadLocal`은 Spring에서 제공하는 `ThreadLocal`의 확장 클래스로, 디버깅을 쉽게 하기 위해 이름을 부여할 수 있도록 설계되었습니다.

- 기본 기능은 `ThreadLocal`과 동일
- 여러 `ThreadLocal`을 사용할 때 이름을 명확히 설정하면 용도를 구분하기 쉬워 **디버깅에 유리**합니다.
