# DB Statistics와 Cardinality 정리

## 핵심 요약

- optimizer가 적절한 join 순서와 index를 고른다.
- stale statistics는 실제 분포 변화를 놓친다.
- estimated rows와 actual rows 비율을 기록한다.

## 개념 설명

cardinality는 조건 결과의 예상 행 수이며 database statistics는 optimizer가 그 row estimate를 계산하는 분포 정보다.

distinct count, null fraction, histogram과 컬럼 상관관계를 이용해 join order와 scan 방식을 고르므로 실제 분포와 statistics가 어긋나면 나쁜 plan이 선택된다.

## 예시

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE status='PAID' AND tenant_id=42;
-- estimated rows=10, actual rows=120000
```

독립 컬럼 가정 때문에 tenant_id와 status가 강하게 상관되면 단일 컬럼 통계가 정확해도 cardinality 오차가 커질 수 있다.

## 면접 답변 예시

> Cardinality는 query 단계가 반환할 것으로 optimizer가 예상한 row 수이고 statistics는 그 추정의 근거입니다. `EXPLAIN ANALYZE`에서 estimated와 actual row가 크게 다르면 join 순서와 scan 방식이 잘못 선택될 수 있습니다. Backfill 뒤에는 `ANALYZE`를 실행하고 tenant와 status처럼 상관된 column은 extended statistics를 검토합니다. Statistics 갱신만으로 plan이 바뀔 수 있어 배포 전후 plan과 latency를 함께 관찰합니다.

## 장점

- row estimate 오차로 plan 문제를 설명한다.

## 단점

- 낮은 sample은 희귀 값을 왜곡한다.

## 주의사항 / 실무 팁

- 대형 backfill 뒤 ANALYZE를 수행한다.
