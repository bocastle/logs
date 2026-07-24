# 결제 승인 timeout 후 상태 조회를 설계하는 법 정리

## 핵심 요약

- 결제 승인 timeout은 실패가 아니라 결과를 확인하지 못한 불확실 상태로 취급한다.
- 승인 요청의 idempotency key와 payment ID를 저장하고 재승인보다 provider 상태 조회를 먼저 수행한다.
- 일정 시간 안에 확정하지 못한 건은 `PENDING`으로 유지하고 reconciliation 작업과 수동 조사 경로로 넘긴다.

## 개념 설명

결제 provider에 승인 요청을 보낸 뒤 timeout이 발생하면 응답만 잃은 것인지 승인도 실패한 것인지 알 수 없다. 이때 내부 상태를 바로 `FAILED`로 바꾸거나 새 승인 요청을 보내면 실제 승인과 원장이 어긋나거나 이중 결제가 생길 수 있다.

요청 전 내부 payment ID와 idempotency key를 내구성 있게 저장한다. Timeout 뒤에는 같은 식별자로 provider의 상태 조회 API를 호출해 `AUTHORIZED`, `FAILED`, `PENDING`을 확인한다. 조회도 실패하면 지수 backoff로 다시 확인하되 사용자에게 성공이나 실패를 추측해 보여 주지 않는다.

## 예시

```text
POST /payments idempotency-key=pay_123 -> timeout
GET /payments/pay_123/status -> AUTHORIZED or FAILED or PENDING
```

Provider가 idempotency key를 지원한다면 같은 key를 재사용하고 새 key로 재승인하지 않는다. 내부 원장과 provider 상태가 끝내 맞지 않는 건은 reconciliation queue에서 자동 또는 수동으로 조정한다.

## 면접 답변 예시

> 결제 승인 timeout은 실패가 아니라 결과를 모르는 상태로 처리하겠습니다. 승인 전에 payment ID와 idempotency key를 저장하고, timeout 뒤에는 새 승인을 보내기보다 같은 식별자로 provider 상태를 조회합니다. 조회도 불가능하면 `PENDING` 상태로 두고 제한된 간격으로 재확인하며 사용자에게 처리 중임을 알립니다. 최종적으로 provider와 내부 원장이 다르면 reconciliation queue와 감사 로그로 조정합니다.

## 장점

- 결과가 불명확한 요청을 추측하지 않아 중복 승인과 승인 누락을 모두 줄일 수 있다.
- Provider와 내부 원장의 불일치를 자동 reconciliation 대상으로 관리할 수 있다.

## 단점

- 새 idempotency key로 바로 재승인하면 이중 승인 가능성이 생긴다.
- Provider 상태 조회 장애가 길어지면 pending 사용자와 운영 조사 건수가 늘어난다.

## 주의사항 / 실무 팁

- Pending 재조회 간격, 최대 자동 대기 시간과 수동 전환 조건을 정한다.
- 승인 요청·조회·webhook이 같은 payment ID와 correlation ID로 연결되는지 확인한다.
