# Kafka Consumer Group Rebalance 정리

## 핵심 요약

- Consumer group rebalance는 member 또는 partition 변화에 따라 partition 소유권을 다시 배분하는 과정이다.
- Stop-the-world에 가까운 eager 재할당과 점진적인 cooperative 재할당의 동작 차이를 이해해야 한다.
- 배포, heartbeat와 `max.poll.interval.ms`를 실제 batch 처리 시간에 맞추고 rebalance 횟수·지속 시간을 관찰한다.

## 개념 설명

Kafka consumer group에서는 한 partition을 group 내 한 consumer가 담당한다. Consumer가 들어오거나 나가고 heartbeat가 끊기거나 partition 수가 바뀌면 coordinator가 소유권을 다시 계산한다.

Eager protocol은 기존 assignment를 모두 revoke한 뒤 다시 나눠 처리 공백이 커질 수 있다. Cooperative assignor는 필요한 partition을 점진적으로 이동해 중단 범위를 줄인다. Stateful consumer는 revoke 전에 offset과 local state를 안전하게 정리하고 새 assignment에서 cache를 다시 준비해야 한다.

## 예시

```text
member-1 owns p0,p1
member-2 joins
rebalance -> member-1:p0, member-2:p1
watch: rebalance_count, revoked_partitions, poll_gap
```

배포 때 instance가 동시에 내려가거나 처리 batch가 `max.poll.interval.ms`를 넘으면 불필요한 rebalance가 반복될 수 있다. Static membership은 짧은 재시작의 churn을 줄일 수 있지만 죽은 member의 ID 관리와 session timeout trade-off가 남는다.

## 면접 답변 예시

> Consumer group rebalance는 member나 partition 변화에 따라 partition 소유권을 다시 나누는 과정입니다. 이때 처리 공백과 lag가 생길 수 있어 cooperative assignor와 static membership으로 불필요한 전체 재할당을 줄일 수 있습니다. `max.poll.interval.ms`는 실제 batch 처리 상한보다 길게 두고 revoke 시 offset과 local state를 정리합니다. 배포 시 rebalance count, duration, poll gap과 partition별 lag를 함께 보겠습니다.

## 장점

- 장애난 member의 partition을 다른 consumer가 이어받아 처리를 계속할 수 있다.
- Consumer 수를 partition 수까지 늘려 병렬 처리량을 확장할 수 있다.

## 단점

- 처리 loop가 `max.poll.interval.ms`를 넘으면 살아 있는 consumer도 제외돼 재조정이 반복될 수 있다.
- Stateful consumer는 partition 이동 때 cache warm-up과 외부 상태 이전 비용이 든다.

## 주의사항 / 실무 팁

- `max.poll.interval.ms`, session timeout과 실제 batch 처리 시간을 함께 조정한다.
- 배포 전략과 assignor 변경 전후의 rebalance duration, revoked partition과 poll gap을 비교한다.
