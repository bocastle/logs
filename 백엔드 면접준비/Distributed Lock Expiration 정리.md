# Distributed Lock Expiration 정리

## 핵심 요약

- 죽은 owner의 lock을 자동 회수한다.
- GC pause가 TTL을 넘으면 살아 있는 owner가 만료된다.
- TTL은 p99 작업 시간과 pause보다 충분히 길게 둔다.

## 개념 설명

distributed lock expiration은 owner 장애 뒤 lock을 회수하기 위해 lock TTL이 지나면 lease 소유권을 무효화하는 규칙이다.

owner가 TTL 안에 작업을 끝내거나 renewal해야 하며, 만료 뒤 새 owner에게 더 큰 fencing token을 발급해 늦은 구 owner의 write를 저장소가 거부해야 한다.

## 예시

```text
acquire resource=invoice:42 ttl=30s -> fencing token=101
TTL expires -> new owner token=102
storage rejects write token=101
```

lock key가 사라졌다는 사실만으로 구 owner 실행이 멈추지는 않으므로 fencing token 없는 TTL lock은 split ownership을 막지 못한다.

## 면접 답변 예시

> Distributed lock의 TTL은 owner가 죽었을 때 lock을 영원히 잡지 않도록 회수하는 lease 장치입니다. 하지만 GC pause나 network 단절로 renewal이 늦으면 기존 owner가 살아 있는 동안 새 owner가 생길 수 있어 TTL만으로 단일 실행을 보장하지 못합니다. Lock service가 단조 증가 fencing token을 발급하고 실제 저장소가 이전 token의 write를 거부할 수 있어야 stale owner를 막을 수 있습니다. TTL은 작업 p99와 pause를 근거로 정하고 owner·token·expiry 갱신을 원자적으로 처리하며, fencing을 검증할 수 없는 외부 side effect에는 별도 멱등 계약을 둡니다.

## 장점

- lease로 무기한 자원 점유를 피한다.

## 단점

- clock skew에 의존하면 만료 판단이 갈린다.

## 주의사항 / 실무 팁

- fencing을 검증할 수 있는 저장소에는 token을 전달하고 외부 API에는 멱등 key를 함께 설계한다.
