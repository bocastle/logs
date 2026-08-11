# PostgreSQL Index Bloat 정리

## 핵심 요약

- 불필요한 page read를 줄여 index scan을 빠르게 한다.
- 재구성 중 기존·신규 index가 함께 있어 disk가 증가한다.
- pgstattuple 표본과 index 사용량을 함께 본다.

## 개념 설명

PostgreSQL index bloat는 update와 delete 뒤 재사용되지 못한 index page 공간 때문에 실제 key 수보다 인덱스가 커진 상태다.

MVCC dead entry, random insert page split, 낮은 재사용률이 원인이며 pgstattuple 또는 page 수 대비 tuple 밀도로 크기를 추정한다.

## 예시

```sql
SELECT * FROM pgstatindex('idx_orders_customer');
REINDEX INDEX CONCURRENTLY idx_orders_customer;
```

REINDEX CONCURRENTLY는 읽기·쓰기를 유지하지만 추가 disk와 더 긴 작업 시간이 필요하므로 bloat 원인부터 줄여야 한다.

## 면접 답변 예시

> PostgreSQL index bloat는 dead entry와 page split 등으로 유효 key 수보다 index가 커진 상태입니다. 먼저 실제 사용량과 `pgstattuple`·page 통계로 bloat를 추정하고 cache miss와 scan I/O에 영향이 있는지 확인합니다. `REINDEX CONCURRENTLY`는 쓰기를 유지할 수 있지만 새 index를 함께 만들 disk와 긴 작업 시간이 필요합니다. 주기적으로 무조건 재구성하기보다 장기 transaction, update pattern과 vacuum 같은 원인을 먼저 줄이겠습니다.

## 장점

- cache에 더 많은 유효 key를 담는다.

## 단점

- 장기 transaction은 dead entry 정리를 막는다.

## 주의사항 / 실무 팁

- REINDEX CONCURRENTLY 전 disk 여유를 계산한다.
