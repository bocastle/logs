# Eventual Consistency UX 보상 정리

## 핵심 요약

- 일시 stale read를 오류로 오인하지 않게 한다.
- 무기한 pending은 사용자 재시도를 유발한다.
- pending SLA와 timeout 문구를 정의한다.

## 개념 설명

eventual consistency UX 보상은 write model과 read model이 아직 수렴하지 않은 동안 pending 상태와 복구 경로를 사용자에게 정확히 보여 주는 설계다.

command 응답의 operation id와 expected version을 UI에 전달하고 projection이 따라올 때까지 optimistic result, polling, reconciliation 상태를 사용한다.

## 예시

```text
POST transfer -> 202 Accepted operation_id=op-91 status=pending
GET operation/op-91 -> applied | compensating | failed
read model version >= expected_version -> 완료 표시
```

성공처럼 먼저 보여 준 뒤 최종 실패를 숨기면 신뢰가 깨지므로 pending과 compensation 결과를 명시적으로 전달한다.

## 면접 답변 예시

> Eventual consistency UX에서는 command가 접수됐다는 사실과 read model까지 반영됐다는 사실을 같은 성공으로 표시하지 않습니다. Write 응답의 operation ID와 expected version으로 pending 상태를 보여 주고 polling이나 push로 `applied`, `compensating`, `failed`를 갱신하겠습니다. Pending SLA를 넘으면 단순 재시도 버튼 대신 현재 처리 상황과 문의·수동 복구 경로를 제공하며 재요청은 idempotency key로 흡수합니다. Optimistic UI가 최종 결과와 다르면 조용히 덮지 않고 보상 결과를 설명하고 reconciliation 실패를 운영 queue와 alert로 연결합니다.

## 장점

- 긴 처리 상태를 operation id로 추적한다.

## 단점

- optimistic UI가 실패 뒤 잘못된 상태를 보여 줄 수 있다.

## 주의사항 / 실무 팁

- idempotency key로 사용자 재시도를 흡수한다.
