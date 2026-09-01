# Workflow Engine과 상태 관리 정리

## 핵심 요약

- 장기 비즈니스 흐름의 실패와 재시작을 체계화한다.
- workflow 정의와 실제 외부 부작용이 어긋나면 중복 실행이 생긴다.
- activity idempotency key를 반드시 설계한다.

## 개념 설명

Workflow Engine은 여러 단계의 장기 작업을 상태, 재시도, 보상, 타임아웃과 함께 관리하는 실행기다.

각 step은 deterministic한 상태 전이를 남기고, 외부 부작용은 activity id와 retry policy로 멱등하게 실행한다.

## 예시

```text
workflow order_refund:
  validate -> cancel_payment -> restore_inventory -> notify
state: step, attempt, last_error, compensation_needed
```

상태를 step 단위로 남기면 중간 실패 후 어디서 재개하거나 보상할지 판단할 수 있다.

## 면접 답변 예시

> Workflow engine은 결제 취소와 재고 복구처럼 오래 걸리는 여러 단계를 상태, retry, timeout과 보상 규칙으로 관리하는 실행기입니다. 중간 상태가 저장되므로 process가 재시작돼도 어디서 이어갈지 알 수 있지만, 외부 API 호출까지 transaction으로 묶어 주는 것은 아닙니다. 각 activity에 안정적인 idempotency key를 주고 retry되어도 같은 부작용이 반복되지 않게 하겠습니다. Step을 지나치게 잘게 쪼개지 않고 운영자가 이해할 business 경계로 나누며 workflow 상태와 실제 business 상태가 어긋나는지도 함께 관찰합니다.

## 장점

- step별 timeout과 retry 정책을 분리할 수 있다.

## 단점

- 상태 저장소 장애가 전체 실행을 막을 수 있다.

## 주의사항 / 실무 팁

- 보상 가능한 step과 불가능한 step을 표시한다.
