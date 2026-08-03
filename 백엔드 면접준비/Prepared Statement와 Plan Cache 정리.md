# Prepared Statement와 Plan Cache 정리

## 핵심 요약

- Prepared statement는 SQL parse·analyze 결과와 parameter 타입을 재사용하고 반복 실행의 plan 비용을 줄일 수 있다.
- PostgreSQL은 custom plan과 generic plan 비용을 비교하므로 bind 값 분포가 치우치면 특정 tenant에서 느린 plan을 선택할 수 있다.
- 대표적인 작은 값·큰 값으로 실제 row와 plan을 비교하고 driver와 server의 statement 수명도 확인한다.

## 개념 설명

Prepared statement는 SQL과 parameter 타입을 이름 있는 statement로 준비해 반복 실행할 수 있게 한다. PostgreSQL은 실행마다 bind 값에 맞는 custom plan을 만들거나 값과 무관한 generic plan을 재사용할 수 있다.

초기 실행의 custom plan 비용과 generic plan의 예상 비용을 비교해 전환할 수 있다. 대부분 tenant는 row가 적지만 하나의 tenant만 매우 크다면 평균적으로 괜찮은 generic plan이 큰 값에서 잘못된 join이나 scan을 선택할 수 있다. Parameter binding과 plan 선택 문제를 분리해서 봐야 한다.

## 예시

```sql
PREPARE find_orders(bigint) AS
SELECT * FROM orders WHERE customer_id = $1;
EXPLAIN (ANALYZE, BUFFERS) EXECUTE find_orders(42);
```

대표 bind별 `EXPLAIN ANALYZE`에서 estimate와 actual row, index 사용을 비교한다. 운영 세션마다 prepared statement가 별도로 존재할 수 있고 connection pool과 driver cache 정책이 수명과 메모리 사용에 영향을 준다.

## 면접 답변 예시

> Prepared statement는 반복 SQL의 parse와 parameter 타입 처리를 재사용하고 plan 비용을 줄일 수 있습니다. PostgreSQL은 bind 값에 맞는 custom plan과 재사용 가능한 generic plan을 비용으로 비교합니다. 값 분포가 치우치면 generic plan이 큰 tenant에 부적합할 수 있어 대표 bind별 actual row와 plan을 확인하겠습니다. Driver·connection pool의 statement cache 수명과 schema 변경 뒤 replan도 함께 관찰하고 custom plan 강제는 문제 query에 제한합니다.

## 장점

- Parameter binding으로 SQL text를 재사용하고 injection 위험을 줄이는 데 도움을 준다.
- 반복 query의 parse·plan CPU를 줄일 수 있다.

## 단점

- Generic plan은 skew가 큰 bind에서 잘못된 scan과 join을 선택할 수 있다.
- DDL과 통계 변화로 invalidation·replan이 몰리는 순간 지연이 튈 수 있다.

## 주의사항 / 실무 팁

- `pg_prepared_statements`, statement 수와 memory, 대표 bind별 plan을 관찰한다.
- Driver cache와 pool connection 수를 곱한 전체 prepared statement 수를 계산한다.
