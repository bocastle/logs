# Hibernate Batch Fetch Size 정리

## 핵심 요약

- N+1의 network round trip을 줄인다.
- 접근하지 않을 proxy까지 함께 로딩할 수 있다.
- 실제 page 크기에 맞춰 16·32·64를 비교한다.

## 개념 설명

Hibernate batch fetch는 여러 lazy proxy 또는 collection을 초기화할 때 id를 IN 조건으로 묶어 N번 SELECT를 더 적은 round trip으로 줄이는 기능이다.

hibernate.default_batch_fetch_size 또는 @BatchSize 값만큼 아직 초기화되지 않은 entity key를 모아 batch fetch SQL을 실행한다.

## 예시

```properties
hibernate.default_batch_fetch_size=32
```
```sql
SELECT * FROM customer WHERE id IN (?, ?, ...);
```

batch size가 너무 크면 IN list parse 비용과 불필요한 row fetch가 늘고 DB parameter 제한에 닿을 수 있다.

## 면접 답변 예시

> Hibernate batch fetch는 여러 lazy proxy나 collection을 초기화할 때 ID를 `IN` 조건으로 묶어 N+1의 round trip 수를 줄이는 기능입니다. Fetch join처럼 root row가 중복되는 pagination 문제를 피할 수 있지만, 접근하지 않을 proxy까지 함께 읽거나 큰 `IN` list를 만들 수 있습니다. Page 크기와 실제 접근 패턴을 기준으로 16, 32, 64 같은 값을 비교하고 query 수뿐 아니라 fetched row와 memory도 보겠습니다. To-one과 collection은 SQL 모양과 효과가 달라 각각 검증합니다.

## 장점

- fetch join이 어려운 여러 collection에 적용할 수 있다.

## 단점

- 큰 IN 목록은 plan과 bind 비용을 높인다.

## 주의사항 / 실무 팁

- SQL count와 fetched row 수를 함께 측정한다.
