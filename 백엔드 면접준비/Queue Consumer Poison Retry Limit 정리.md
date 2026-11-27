# Queue Consumer Poison Retry Limit 정리

## 핵심 요약

- poison message 하나가 consumer를 계속 점유하는 것을 막는다.
- 일시 장애를 너무 빨리 DLQ로 보내면 복구 후 자동 처리 기회를 잃는다.
- validation 오류와 timeout 오류의 retry 횟수를 다르게 둔다.

## 개념 설명

Queue Consumer Poison Retry Limit은 계속 실패하는 메시지를 무한 재시도하지 않도록 최대 재시도 횟수와 격리 기준을 정하는 정책이다.

오류 유형별 retryable 여부를 나누고, attempt가 한도를 넘으면 DLQ나 quarantine으로 이동시킨다.

## 예시

```text
attempt=1..3 retry with backoff
attempt=4 -> DLQ(reason=VALIDATION_ERROR)
watch: retry_count, dlq_count, retry_loop_count
```

retry limit은 메시지를 포기하는 기준이 아니라 정상 흐름을 보호하기 위한 격리 기준이다.

## 면접 답변 예시

> Poison message retry limit은 계속 실패하는 한 건이 consumer를 점유하지 않도록 오류 유형별 재시도 횟수와 격리 기준을 정하는 정책입니다. Schema나 validation 오류는 같은 payload로 반복해도 낫지 않으므로 바로 quarantine하고 timeout 같은 일시 오류만 backoff·jitter와 최대 횟수를 적용하겠습니다. 한도를 넘기면 원본 payload, message ID, error와 attempt를 보존해 DLQ로 옮기고 source offset ack와 격리 저장 사이에서 유실이나 무한 중복이 생기지 않게 처리 순서를 검증합니다. DLQ에는 owner, 알림과 수정 후 제한 replay 절차를 두며 count뿐 아니라 oldest age와 같은 message의 retry loop를 관찰합니다.

## 장점

- retry와 DLQ 사이의 실패 수명 주기를 명확히 만든다.

## 단점

- 한도가 너무 높으면 retry storm이 생긴다.

## 주의사항 / 실무 팁

- exponential backoff와 jitter를 적용한다.
