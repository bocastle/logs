# Soft delete와 archive table을 나누는 기준 정리

## 핵심 요약

- Soft delete는 같은 테이블에 삭제 상태를 남겨 짧은 기간 복구하기 쉽고, archive는 오래된 데이터를 hot query 경로에서 분리한다.
- 복구 기간, 법적 삭제 요구, 참조 무결성과 운영 조회 빈도를 기준으로 두 단계를 나눈다.
- Archive는 복사 성공 확인, 원본 삭제와 재실행 멱등성을 포함한 별도 migration으로 운영한다.

## 개념 설명

Soft delete는 원본 row에 `deleted_at`을 기록해 일반 조회에서 제외하는 방식이다. 빠른 복원과 기존 참조 유지에는 유리하지만 table과 index에는 계속 남는다. Archive는 복구 기간이 지난 row를 별도 table이나 storage로 옮겨 active 데이터 경로를 줄인다.

두 방식은 대안이라기보다 단계로 함께 사용할 수 있다. 일정 기간 soft delete 상태로 보관한 뒤 archive하고, 보존 기간이 끝나면 영구 삭제한다. 개인정보 삭제 요청은 archive에도 동일하게 적용돼야 하며 운영·감사 도구가 어느 저장소를 조회할지도 명시한다.

## 예시

```sql
CREATE INDEX idx_orders_active ON orders(customer_id)
WHERE deleted_at IS NULL;
INSERT INTO orders_archive SELECT * FROM orders WHERE deleted_at < :cutoff;
```

Partial index는 active 조회를 줄이지만 filter 누락을 막아 주지는 않는다. Archive 작업은 같은 row를 중복 복사해도 안전한 key를 사용하고 row count·checksum을 확인한 뒤 원본을 삭제한다. 참조 table과 동시 쓰기의 경계도 따로 설계해야 한다.

## 면접 답변 예시

> Soft delete는 같은 table에 삭제 시각을 남겨 짧은 기간 복구하기 좋고, archive는 오래된 row를 hot query에서 분리하는 방식입니다. 복구 기간 동안 soft delete로 두었다가 row count와 checksum을 검증하며 archive한 뒤 원본을 제거할 수 있습니다. 일반 조회는 `deleted_at IS NULL`을 repository 경계에서 강제하고 partial index 사용을 확인합니다. 법적 삭제와 참조 무결성은 archive 저장소까지 같은 정책을 적용해야 합니다.

## 장점

- Archive table은 active index 크기와 vacuum 대상 범위를 줄일 수 있다.
- Soft delete는 사용자 실수와 짧은 복구 요청을 빠르게 되돌리기 쉽다.

## 단점

- Archive 이동 중 동시 쓰기와 참조 관계를 놓치면 데이터가 누락되거나 깨질 수 있다.
- 두 저장소를 함께 조회하는 감사·운영 도구와 개인정보 삭제 경로가 복잡해진다.

## 주의사항 / 실무 팁

- Archive 전후 row count와 checksum을 검증하고 재실행해도 중복되지 않는 key를 둔다.
- 복구 가능 기간, archive 보존 기간과 영구 삭제 시점을 데이터 분류별로 명시한다.
