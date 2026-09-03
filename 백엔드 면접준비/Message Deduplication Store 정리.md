# Message Deduplication Store 정리

## 핵심 요약

- 동시에 들어온 같은 메시지 중 한 실행만 진행 상태를 획득하게 할 수 있다.
- `INPROGRESS` 만료를 무조건 재실행하면 외부 side effect가 중복될 수 있다.
- message id와 business key 중 실제 중복 기준을 먼저 고른다.

## 개념 설명

Message Deduplication Store는 메시지 id나 비즈니스 key를 저장해 중복 소비 여부를 판단하는 저장소다.

외부 side effect가 있으면 key 존재 여부만 저장하지 말고 `INPROGRESS`와 `COMPLETE`, 실행 중 만료 시각을 관리한다. 재시도는 완료 결과를 재사용하고 만료된 실행만 복구한다.

## 예시

```sql
INSERT INTO message_dedup(message_key, status, in_progress_until, expires_at)
VALUES (:key, 'INPROGRESS', now() + interval '2 minutes', now() + interval '7 days')
ON CONFLICT DO NOTHING
RETURNING message_key;

-- row가 반환돼 claim을 획득한 실행만 외부 API를 호출
-- 외부 호출에는 message_key를 idempotency key로 전달
UPDATE message_dedup
SET status = 'COMPLETE', in_progress_until = NULL
WHERE message_key = :key AND status = 'INPROGRESS';
```

`RETURNING` 결과가 없으면 기존 상태를 읽고 외부 호출을 건너뛴다. 외부 호출 성공 뒤 `COMPLETE` 저장 전에 crash가 나면 재호출될 수 있으므로 대상 시스템도 같은 멱등 key를 받아야 한다.

## 면접 답변 예시

> 실행 중 timeout과 오래된 record를 서로 다른 만료 정책으로 관리할 수 있다. key 충돌이 있으면 정상 메시지를 다른 처리의 재시도로 오판한다. `INPROGRESS` 만료, `COMPLETE` 재사용, payload 불일치 경로를 테스트한다.

## 장점

- 완료 결과를 저장하면 재시도 요청에 같은 응답을 돌려줄 수 있다.

## 단점

- `COMPLETE` 보관 기간이 replay window보다 짧으면 오래된 중복을 놓친다.

## 주의사항 / 실무 팁

- 외부 API와 동일한 멱등 key 계약을 사용한다.
