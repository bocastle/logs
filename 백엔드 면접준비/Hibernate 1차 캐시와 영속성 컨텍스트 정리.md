# Hibernate 1차 캐시와 영속성 컨텍스트 정리

## 핵심 요약

- 같은 entity의 중복 SELECT를 줄인다.
- 큰 batch에서 managed entity가 쌓여 메모리를 사용한다.
- batch 처리에서는 주기적으로 flush와 clear를 한다.

## 개념 설명

Hibernate 1차 캐시는 영속성 컨텍스트 내부에서 entity type과 id별로 managed instance를 보관하는 영속성 컨텍스트 범위 cache다. 일반적인 transaction-scoped EntityManager에서는 transaction과 범위가 거의 일치하지만 항상 같은 개념은 아니다.

같은 id를 다시 조회하면 DB 대신 동일 object를 반환해 entity identity를 보장하고, snapshot과 쓰기 지연으로 unit of work를 구성한다.

## 예시

```java
Order a = em.find(Order.class, 42L);
Order b = em.find(Order.class, 42L);
assert a == b; // 1차 캐시의 동일 instance
```

1차 캐시는 다른 transaction이나 process와 공유되지 않으며 bulk SQL 뒤에는 영속성 컨텍스트가 stale할 수 있다.

## 면접 답변 예시

> Hibernate 1차 캐시는 영속성 컨텍스트 안에서 같은 entity type과 id를 동일한 managed instance로 관리하는 cache입니다. 이 identity map을 바탕으로 dirty checking과 쓰기 지연이 동작하고, 같은 id를 다시 찾을 때 불필요한 SELECT도 줄일 수 있습니다. 다만 bulk DML이나 외부 update는 관리 중인 instance에 자동 반영되지 않아 stale 상태가 남을 수 있습니다. Batch 처리에서는 주기적으로 flush와 clear를 하고, process 간 공유되는 2차 cache와 범위를 혼동하지 않겠습니다.

## 장점

- transaction 안에서 entity identity를 보장한다.

## 단점

- 외부 update를 자동으로 반영하지 않는다.

## 주의사항 / 실무 팁

- 외부 변경이 있으면 refresh를 명시한다.
