# Partial Index 정리

## 핵심 요약

- 관심 row만 저장해 index 크기와 cache·쓰기 비용을 줄인다.
- Predicate 밖의 조회에는 도움이 되지 않고 planner가 query 조건의 포함 관계를 증명할 수 있어야 한다.
- 실제 parameterized query를 `EXPLAIN`하고 상태 분포가 바뀌면 selectivity를 다시 본다.

## 개념 설명

partial index는 table 전체가 아니라 WHERE predicate를 만족하는 행만 저장하는 인덱스다.

query predicate가 index predicate를 논리적으로 포함할 때 optimizer가 사용할 수 있다. PostgreSQL은 단순 부등식의 포함 관계는 인식하지만 일반적인 정리 증명까지 하지는 않는다.

## 예시

```sql
CREATE INDEX idx_orders_open
ON orders(tenant_id, created_at DESC)
WHERE status = 'OPEN';
```

parameterized 조건이나 표현식이 predicate와 다르면 planner가 포함 관계를 증명하지 못해 partial index를 쓰지 않을 수 있다.

## 면접 답변 예시

> Partial index는 table 전체가 아니라 predicate를 만족하는 row만 저장하는 index입니다. 열린 주문처럼 전체 중 일부를 자주 조회할 때 index 크기와 쓰기 비용을 줄일 수 있습니다. 다만 query 조건이 index predicate를 포함한다고 planner가 증명할 수 있어야 사용되며 parameter와 표현 방식 때문에 선택되지 않을 수 있습니다. 실제 query를 `EXPLAIN`하고 상태 분포가 변할 때 index 크기와 selectivity를 다시 확인합니다.

## 장점

- 관심 없는 row의 index 유지 비용을 피하고 active row에만 조건부 uniqueness를 적용할 수 있다.

## 단점

- Predicate가 query와 논리적으로 맞아도 planner가 표현을 증명하지 못하면 사용되지 않을 수 있다.
- 상태 분포가 바뀌면 작은 index라는 전제가 무너질 수 있다.

## 주의사항 / 실무 팁

- Production과 같은 prepared query를 `EXPLAIN`해 index 선택을 검증한다.
- Predicate 대상 row 비율과 index 크기를 주기적으로 확인한다.
