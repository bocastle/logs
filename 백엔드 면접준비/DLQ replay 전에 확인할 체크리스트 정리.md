# DLQ replay 전에 확인할 체크리스트 정리

## 핵심 요약

- DLQ replay는 queue를 비우는 작업이 아니라 실패 원인을 고친 뒤 메시지를 안전하게 다시 처리하는 복구 절차다.
- 재처리 전에 consumer의 현재 코드와 schema 호환성, 멱등 키, 외부 부작용과 재실패 경로를 확인한다.
- 전체를 한 번에 넣지 않고 대표 표본과 작은 batch부터 속도를 제한해 진행한다.

## 개념 설명

Dead letter queue에는 반복 실패하거나 처리할 수 없어 본 흐름에서 격리된 메시지가 모인다. Replay는 원인을 분류하고 수정한 뒤 선택한 메시지를 다시 consumer 흐름으로 보내는 운영 작업이다.

먼저 schema 오류, 데이터 문제, 일시적인 downstream 장애처럼 원인별로 묶는다. 코드가 수정됐는지 확인하고 결제·알림·적립 같은 부작용이 event ID로 멱등한지 검증한다. 재실패 메시지가 원래 DLQ를 무한 순환하지 않도록 replay 식별자와 별도 quarantine 기준도 둔다.

## 예시

```text
DLQ -> sample 100 -> fix consumer -> replay topic
rate=200 msg/min
on failure -> quarantine again
watch: replay_success, duplicate_effect, dlq_return_count
```

표본 100개가 정상 처리됐다고 바로 전체 속도를 올리지 않는다. Downstream 지연, 중복 효과와 DLQ 재유입률을 보며 batch를 확대하고 언제 중단할지도 미리 정한다.

## 면접 답변 예시

> DLQ replay는 메시지를 다시 넣기 전에 실패 원인과 수정 여부, schema 호환성, consumer 멱등성을 확인하는 복구 절차입니다. 결제나 알림 같은 외부 부작용이 중복되지 않는지 표본으로 검증하고 작은 batch부터 rate limit을 걸어 진행합니다. Replay 중에는 성공률뿐 아니라 downstream 지연과 DLQ 재유입률을 보며 중단 조건을 적용합니다. 어떤 메시지를 누가 언제 재처리했는지도 감사 로그에 남깁니다.

## 장점

- 장애 원인별로 replay 범위와 속도를 나눠 위험한 메시지만 별도 격리할 수 있다.
- 격리된 메시지를 폐기하지 않고 수정된 consumer로 회복할 수 있다.

## 단점

- 원인 수정 전 replay하면 같은 오류로 DLQ가 다시 빠르게 찬다.
- 큰 batch는 broker뿐 아니라 데이터베이스와 외부 API에 평소보다 큰 부하를 줄 수 있다.

## 주의사항 / 실무 팁

- Replay 전 consumer의 idempotency key와 외부 side effect의 중복 방지를 검증한다.
- Replay ID, 원래 event ID, 처리 결과와 재실패 위치를 감사 가능한 형태로 남긴다.
