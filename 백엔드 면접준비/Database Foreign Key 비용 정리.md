# Database Foreign Key 비용 정리

## 핵심 요약

- 고아 row를 database 경계에서 차단한다.
- hot parent key에서 concurrent write가 경합한다.
- 모든 child FK column에 필요한 index를 검토한다.

## 개념 설명

foreign key는 child key가 parent row를 참조하도록 referential integrity를 강제하며 write마다 존재 확인과 동시성 lock 비용을 만든다.

child INSERT는 parent key를 index로 조회하고 parent DELETE·UPDATE는 참조 child를 확인하므로 양쪽 key index와 lock ordering이 중요하다.

## 예시

```sql
ALTER TABLE order_items ADD CONSTRAINT fk_order
FOREIGN KEY(order_id) REFERENCES orders(id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

child foreign key index가 없으면 parent 삭제가 child table을 scan해 lock 보유 시간과 latency를 크게 늘릴 수 있다.

## 면접 답변 예시

> Foreign key는 여러 writer가 같은 referential integrity를 지키게 하지만 child insert마다 parent 존재 확인과 parent 변경 시 child 참조 검사가 필요합니다. Parent key에는 unique index가 필요하고 PostgreSQL처럼 child FK index를 자동 생성하지 않는 DB에서는 delete와 update 경로에 맞춰 직접 index를 검토하겠습니다. `CASCADE`, `RESTRICT` 의미와 transaction별 parent·child 변경 순서를 통일해 lock 경합과 deadlock을 줄입니다. 대량 적재에서는 constraint 추가와 validation의 lock, 처리 시간과 throughput 영향을 실제 data 규모로 측정합니다.

## 장점

- 여러 writer가 같은 referential integrity를 공유한다.

## 단점

- child index 누락은 parent 변경을 느리게 한다.

## 주의사항 / 실무 팁

- parent·child 변경 순서를 transaction마다 통일한다.
