# Optimistic Force Increment Lock 정리

## 핵심 요약

- 읽기 기반 의사결정의 동시 변경을 탐지한다.
- 읽기만 해도 version UPDATE가 발생한다.
- aggregate 경계가 필요한 곳에만 적용한다.

## 개념 설명

OPTIMISTIC_FORCE_INCREMENT는 JPA가 entity를 읽을 때 @Version 값을 강제로 증가시켜 그 entity에 의존한 후속 쓰기 충돌을 드러내는 lock mode다.

provider가 flush나 commit 과정에서 version UPDATE를 실행하고 다른 transaction이 이전 version으로 저장하면 optimistic lock exception을 발생시킨다.

## 예시

```java
Order order = em.find(Order.class, id,
    LockModeType.OPTIMISTIC_FORCE_INCREMENT);
// UPDATE orders SET version=version+1 WHERE id=? AND version=?
```

부모 aggregate를 직접 바꾸지 않아도 자식 변경의 동시성을 부모 version에 연결할 때 유용하지만 불필요한 version write는 경합을 키운다.

## 면접 답변 예시

> `OPTIMISTIC_FORCE_INCREMENT`는 entity를 읽은 transaction이 직접 필드를 바꾸지 않더라도 version을 증가시켜 그 aggregate에 의존한 동시 작업의 충돌을 드러내는 lock mode입니다. DB row lock을 오래 잡는 방식은 아니며 provider가 flush나 commit 과정에서 version 조건이 맞는지 확인합니다. 부모 version으로 자식 변경까지 같은 동시성 경계에 묶을 때 유용하지만 hot aggregate에서는 불필요한 update와 conflict가 늘 수 있습니다. `OptimisticLockException` 재시도 안의 외부 부작용은 멱등하게 만들고 entity별 충돌률을 보고 적용 범위를 제한하겠습니다.

## 장점

- aggregate 자식 변경을 부모 version에 연결한다.

## 단점

- hot aggregate에서 optimistic conflict가 급증한다.

## 주의사항 / 실무 팁

- OptimisticLockException 재시도는 멱등하게 만든다.
