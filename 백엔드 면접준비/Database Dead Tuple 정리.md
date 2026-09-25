# Database Dead Tuple 정리

## 핵심 요약

- dead tuple 수로 vacuum 압력을 관찰한다.
- 정리 지연은 table·index bloat를 만든다.
- n_dead_tup과 transaction age를 함께 본다.

## 개념 설명

dead tuple은 PostgreSQL MVCC에서 UPDATE나 DELETE 뒤 새 transaction에는 보이지 않지만 아직 물리 page에 남아 있는 이전 row version이다.

VACUUM은 모든 active snapshot보다 오래된 dead tuple을 재사용 가능 상태로 표시하며 오래 열린 transaction은 xmin horizon을 붙잡아 정리를 막는다.

## 예시

```sql
SELECT relname, n_live_tup, n_dead_tup, last_autovacuum
FROM pg_stat_user_tables ORDER BY n_dead_tup DESC;
```

UPDATE는 기존 tuple을 dead로 만들고 새 version을 추가하므로 쓰기 집중 table에서 autovacuum이 처리율을 따라가지 못하면 bloat가 쌓인다.

## 면접 답변 예시

> PostgreSQL dead tuple은 MVCC에서 update나 delete 후 새 transaction에는 보이지 않지만 page에 남아 있는 이전 row version입니다. Autovacuum이 이를 재사용 가능하게 만들지만 오래 열린 transaction이나 `idle in transaction` session이 xmin horizon을 붙잡으면 정리가 진행되지 않습니다. `n_dead_tup`만 보지 않고 transaction age, table·index 크기와 autovacuum 처리율을 함께 확인하겠습니다. Write-heavy table은 threshold와 scale factor를 별도 조정하되 vacuum I/O가 foreground query를 압박하지 않는지도 관찰합니다.

## 장점

- 재사용 가능한 page 공간을 회복한다.

## 단점

- idle in transaction이 xmin을 오래 유지한다.

## 주의사항 / 실무 팁

- idle in transaction timeout을 설정한다.
