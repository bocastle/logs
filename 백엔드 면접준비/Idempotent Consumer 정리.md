# Idempotent Consumer 정리

## 핵심 요약

- DB 결과와 중복 기록을 하나의 commit 경계로 관리할 수 있다.
- dedup row만 먼저 commit하면 crash 뒤 실제 처리가 영구 누락될 수 있다.
- DB side effect와 dedup 기록을 같은 트랜잭션에 넣는다.

## 개념 설명

Idempotent Consumer는 같은 메시지를 여러 번 받아도 비즈니스 결과가 한 번 처리된 것처럼 유지되게 만드는 소비자 설계다.

DB 변경이라면 dedup row와 비즈니스 결과를 같은 트랜잭션에 저장해야 한다. 외부 side effect는 대상 API의 idempotency key나 별도 상태 머신으로 보호한다.

## 예시

```sql
BEGIN;
WITH claimed AS (
  INSERT INTO processed_messages(message_id, processed_at)
  VALUES (:message_id, now())
  ON CONFLICT DO NOTHING
  RETURNING message_id
)
UPDATE orders
SET status = 'PAID'
WHERE id = :order_id
  AND EXISTS (SELECT 1 FROM claimed);
COMMIT;
```

dedup insert와 DB side effect가 같은 트랜잭션이면 중간 crash에서 둘 다 rollback된다. 외부 호출은 이 원자성에 포함되지 않으므로 같은 idempotency key로 재시도해야 한다.

## 면접 답변 예시

> Idempotent consumer는 broker가 같은 메시지를 다시 전달해도 business 결과를 한 번 처리한 것처럼 유지하는 설계입니다. DB 변경이라면 message ID claim과 business update를 같은 transaction에 넣어 둘 다 commit되거나 rollback되게 합니다. 외부 API 호출은 그 transaction에 포함되지 않으므로 같은 idempotency key를 대상 system에도 전달하거나 별도 상태 machine으로 보호합니다. Commit 실패, timeout과 replay를 장애 주입 test로 확인하겠습니다.

## 장점

- broker 재전달과 consumer 재시작이 비즈니스 중복으로 이어지는 것을 줄인다.

## 단점

- 잘못 고른 business key는 서로 다른 작업을 같은 요청으로 오판한다.

## 주의사항 / 실무 팁

- 외부 호출에는 동일한 idempotency key를 전달한다.
