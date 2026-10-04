# Lease Renewal 실패 처리 정리

## 핵심 요약

- 소유권 불확실 상태의 side effect를 조기에 멈춘다.
- 너무 빠른 중단은 일시 network jitter에 민감하다.
- TTL·renew 주기·safety margin을 명시한다.

## 개념 설명

lease renewal 실패는 owner가 TTL 연장 확인을 받지 못해 자원 소유권이 계속 유효한지 알 수 없는 상태다.

연속 renewal 실패 또는 남은 TTL이 safety margin 아래로 내려가면 작업을 중단하고, 저장소는 fencing token으로 이후 stale write를 거부한다.

## 예시

```text
lease ttl=30s renewal every=10s
renewal timeout at t=20s, t=24s
remaining TTL < 5s -> stop side effects, token=91 폐기
```

network partition에서는 renewal 요청이 실제 성공했는지 불확실하므로 local 성공 추정 대신 lease store 응답과 token 검증을 따른다.

## 면접 답변 예시

> Lease renewal에 실패하면 owner는 TTL이 남아 보여도 소유권을 확신할 수 없는 상태로 봐야 합니다. Renewal 주기와 timeout, safety margin을 정하고 남은 시간이 margin 아래로 내려가면 새 side effect를 시작하지 않고 작업을 중단하겠습니다. Network partition에서는 renewal 요청이 실제 반영됐는지 모를 수 있어 local 추정이 아니라 lease store의 확인과 fencing token을 따릅니다. Token을 검증할 수 없는 외부 작업에는 idempotency key를 쓰고 renewal loop 자체의 정지와 지연도 main 작업과 별도로 감시합니다.

## 장점

- TTL 만료 뒤 새 owner가 작업을 인수한다.

## 단점

- 작업 취소가 불가능하면 외부 side effect가 계속될 수 있다.

## 주의사항 / 실무 팁

- renewal loop health를 main 작업과 독립 감시한다.
