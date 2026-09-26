# Distributed Semaphore 정리

## 핵심 요약

- 외부 API와 DB에 대한 cluster 동시성을 제한한다.
- permit 누수는 전체 처리량을 고갈시킨다.
- permit에 owner와 lease token을 저장한다.

## 개념 설명

distributed semaphore는 여러 process가 공유 자원에 동시에 진입할 수 있는 permit 수를 cluster 전체에서 제한하는 coordination primitive다.

permit 획득과 lease 만료를 원자적으로 기록하고 사용이 끝나면 반환해 bounded concurrency를 유지한다.

## 예시

```text
semaphore=db-migration permits=3
worker-7 acquire -> permit token=88 ttl=60s
active permits=3 -> next worker waits
```

owner crash 뒤 permit 회수를 위해 TTL을 쓰면 늦은 owner가 계속 실행할 수 있으므로 permit token 검증과 idempotency가 필요하다.

## 면접 답변 예시

> Distributed semaphore는 cluster 전체에서 외부 API나 DB에 동시에 진입할 수 있는 작업 수를 permit으로 제한하는 장치입니다. Permit 획득과 owner·lease token 저장을 원자적으로 처리하고 release는 같은 token을 가진 owner만 할 수 있게 하겠습니다. Crash 회수를 위한 TTL이 만료돼도 기존 작업이 실제로 멈추는 것은 아니어서 잠시 동시성 상한을 넘을 수 있습니다. 보호 대상이 token을 검증할 수 있으면 fencing을 사용하고 그렇지 않으면 idempotency와 보수적인 TTL을 더하며 사용 permit, 대기 시간과 만료 회수를 관찰합니다.

## 장점

- mutex보다 여러 안전 작업을 병렬 허용한다.

## 단점

- 불공정 queue는 특정 worker starvation을 만든다.

## 주의사항 / 실무 팁

- 대기 시간과 사용 permit 수를 측정한다.
