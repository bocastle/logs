# Database Covering Index 한계 정리

## 핵심 요약

- 선택 query의 table page read를 줄인다.
- 넓은 leaf tuple은 cache 효율을 낮춘다.
- 실제 projection에 필요한 좁은 column만 INCLUDE한다.

## 개념 설명

covering index는 query filter·정렬·출력 column을 index key와 INCLUDE에 담아 table lookup을 줄이는 index 설계다.

key column은 탐색과 ordering에 참여하지만 INCLUDE column은 leaf payload로만 저장되어 조회를 덮는 대신 모든 변경에서 유지된다.

## 예시

```sql
CREATE INDEX idx_orders_customer_created
ON orders(customer_id, created_at DESC)
INCLUDE(status, total_amount);
```

covering index라도 MVCC visibility 정보가 부족하면 heap을 방문하며 INCLUDE가 많을수록 index 크기와 write amplification이 커진다.

## 면접 답변 예시

> Covering index는 filter와 sort key에 projection column을 `INCLUDE`해 table page lookup을 줄이는 설계입니다. Included column은 ordering에 참여하지 않지만 leaf payload와 모든 관련 update 비용을 늘립니다. PostgreSQL에서는 필요한 column이 모두 있어도 visibility map이 충분하지 않으면 MVCC 확인을 위해 heap을 방문하므로 index-only scan 이름만 믿으면 안 됩니다. `EXPLAIN ANALYZE`의 heap fetches, index 크기와 write p99를 전후 비교해 실제 이득이 있는 좁은 projection만 포함하겠습니다.

## 장점

- 정렬과 projection을 한 index에서 처리한다.

## 단점

- included column update도 index write를 만든다.

## 주의사항 / 실무 팁

- index 크기와 write p99를 추가 전후 비교한다.
