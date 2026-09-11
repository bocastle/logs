# Transactional Inbox 정리

## 핵심 요약

- at-least-once delivery의 중복 업무 처리를 막는다.
- message_id가 안정적이지 않으면 정상 이벤트를 중복으로 오인한다.
- producer가 전역적으로 안정된 message_id를 발급한다.

## 개념 설명

transactional inbox는 consumer가 받은 message_id와 업무 변경을 같은 local transaction에 저장해 중복 처리를 막는 패턴이다.

inbox table의 message_id에 unique constraint를 두고 INSERT가 성공한 경우에만 side effect를 적용하면 broker redelivery를 멱등하게 흡수한다.

## 예시

```sql
INSERT INTO inbox_messages(message_id, consumed_at)
VALUES (:message_id, now()) ON CONFLICT DO NOTHING;
-- inserted row가 있을 때만 업무 UPDATE
```

inbox commit 뒤 broker ack가 실패해도 다음 redelivery는 unique constraint에서 중복으로 판정된다.

## 면접 답변 예시

> Transactional inbox는 consumer가 message ID 기록과 local business update를 같은 DB transaction에 넣어 redelivery 중복을 흡수하는 패턴입니다. Inbox의 unique constraint에서 처음 insert한 실행만 업무 변경을 수행하면 commit 뒤 broker ack가 실패해도 다음 delivery는 이미 처리된 것으로 판단할 수 있습니다. 이 보장은 local database 안에만 적용되므로 외부 API에는 같은 idempotency key를 전달하거나 outbox 같은 별도 경계를 사용하겠습니다. Inbox retention은 broker의 최대 replay 기간보다 길게 두고 stable message ID가 실제 업무 중복 기준과 맞는지도 확인합니다.

## 장점

- 업무 상태와 소비 기록을 원자적으로 묶는다.

## 단점

- inbox 보관량이 계속 늘 수 있다.

## 주의사항 / 실무 팁

- inbox retention을 broker replay 기간보다 길게 둔다.
