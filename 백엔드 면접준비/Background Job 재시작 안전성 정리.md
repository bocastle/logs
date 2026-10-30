# Background Job 재시작 안전성 정리

## 핵심 요약

- worker 장애 후 수동 개입 없이 복구하기 쉽다.
- checkpoint 갱신과 실제 쓰기가 원자적이지 않으면 누락이 생긴다.
- 작업 단위는 재시도 가능한 작은 chunk로 나눈다.

## 개념 설명

Background Job 재시작 안전성은 worker가 중간에 죽어도 같은 작업을 중복·누락 없이 이어 갈 수 있게 만드는 성질이다.

작업은 checkpoint, idempotency key, lease, heartbeat를 남기고 재시작 시 완료된 범위 이후부터 재개한다.

## 예시

```text
job_id=backfill_7
checkpoint last_id=102400
lease_owner=worker-3 heartbeat=10:00:05
restart -> resume after last_id
```

checkpoint가 있어야 재시작 후 처음부터 다시 돌며 중복 쓰기를 만드는 일을 줄일 수 있다.

## 면접 답변 예시

> Background job의 restart safety는 worker가 어느 지점에서 죽어도 작은 chunk를 다시 실행해 같은 결과로 수렴하게 만드는 성질입니다. Result write와 checkpoint를 같은 transaction에 묶을 수 있으면 가장 단순하고, 그렇지 않다면 chunk key로 idempotent upsert한 뒤 checkpoint를 전진시키겠습니다. Lease와 heartbeat는 중복 worker를 줄이지만 TTL 만료가 기존 실행을 멈추게 하지는 않아 side effect도 멱등해야 합니다. Crash point별 재실행, heartbeat 지연과 lease steal을 test하고 checkpoint가 가리키는 결과가 실제로 저장됐는지 검증합니다.

## 장점

- 긴 작업을 작은 단위로 나눠 진행률을 볼 수 있다.

## 단점

- lease 만료가 짧으면 정상 작업을 다른 worker가 빼앗을 수 있다.

## 주의사항 / 실무 팁

- checkpoint와 결과 쓰기의 순서를 테스트한다.
