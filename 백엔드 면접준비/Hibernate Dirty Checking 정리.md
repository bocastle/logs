# Hibernate Dirty Checking 정리

## 핵심 요약

- 도메인 메서드 변경을 transaction commit에 자연스럽게 반영한다.
- 의도하지 않은 setter 호출도 UPDATE를 만든다.
- transaction별 managed entity 수를 제한한다.

## 개념 설명

Hibernate dirty checking은 영속성 컨텍스트가 관리 entity의 현재 값과 최초 snapshot을 비교해 flush 시점에 UPDATE를 생성하는 기능이다.

transaction 안에서 managed entity를 변경하면 명시적 save 없이도 flush 전에 snapshot 차이를 계산하고 변경 SQL을 실행한다.

## 예시

```java
Order order = entityManager.find(Order.class, id);
order.changeStatus(PAID);
// transaction commit -> dirty checking -> UPDATE
```

큰 영속성 컨텍스트는 snapshot 비교와 메모리 비용을 키우며 bulk update는 1차 캐시 상태를 자동으로 맞추지 않는다.

## 면접 답변 예시

> Hibernate dirty checking은 영속성 context가 managed entity의 snapshot과 현재 값을 비교해 flush 때 `UPDATE`를 만드는 기능입니다. Domain method만 호출해도 transaction commit에 변경이 반영되지만 관리 entity가 많으면 snapshot memory와 비교 비용이 커집니다. Bulk DML은 1차 cache 상태를 자동으로 맞추지 않으므로 뒤에 `clear()`나 `refresh()`를 명시해야 합니다. SQL log와 update count로 의도하지 않은 setter와 UPDATE를 확인하겠습니다.

## 장점

- 변경 감지를 unit of work로 묶는다.

## 단점

- 관리 entity 수가 많으면 snapshot 비용이 커진다.

## 주의사항 / 실무 팁

- bulk DML 뒤 clear 또는 refresh를 명시한다.
