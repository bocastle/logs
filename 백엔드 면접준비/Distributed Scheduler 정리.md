# Distributed Scheduler 정리

## 핵심 요약

- 예약 작업을 노드 장애와 무관하게 이어 갈 수 있다.
- lease 갱신 실패가 중복 실행으로 이어질 수 있다.
- lease TTL은 최대 heartbeat 지연보다 충분히 길게 둔다.

## 개념 설명

Distributed Scheduler는 여러 노드 중 하나 또는 정해진 수의 worker가 예약 작업을 실행하도록 조정하는 시스템이다.

작업 정의, next run time, lease owner, fencing token을 저장하고 실행자는 lease를 획득한 뒤 heartbeat로 소유권을 갱신한다.

## 예시

```text
job daily-report due=10:00
worker-2 acquires lease token=42 ttl=60s
heartbeat extends lease until finished
```

fencing token을 결과 쓰기에도 사용해야 오래된 worker가 늦게 끝나며 최신 결과를 덮어쓰는 일을 막을 수 있다.

## 면접 답변 예시

> Distributed scheduler는 여러 node 중 정해진 worker만 due job을 실행하도록 next run, lease owner와 실행 이력을 관리하는 시스템입니다. Lease와 heartbeat는 중복을 줄이지만 renewal failure 뒤 old worker가 계속 실행할 수 있어 result store가 fencing token을 검증하거나 job 자체가 멱등해야 합니다. Clock skew와 scheduler store 장애 때 missed run을 skip할지 catch up할지도 업무별로 정하겠습니다. Missed, duplicate와 late run을 별도 지표로 보고 recovery drill에서 같은 schedule을 두 node가 잡는 상황을 test합니다.

## 장점

- 작업 실행 이력과 실패를 중앙에서 추적할 수 있다.

## 단점

- clock skew가 크면 due time 판단이 어긋난다.

## 주의사항 / 실무 팁

- 작업 결과에는 fencing token 검사를 넣는다.
