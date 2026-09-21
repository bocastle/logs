# Kafka Partition Assignment Strategy 정리

## 핵심 요약

- consumer별 partition 수를 더 균등하게 만들 수 있다.
- 전략 변경 자체가 rolling 배포와 호환성 이슈를 만들 수 있다.
- 현재 moved partition 수와 rebalance duration을 먼저 측정한다.

## 개념 설명

Kafka Partition Assignment Strategy는 consumer group의 partition을 consumer에게 어떻게 배분할지 정하는 전략이다.

range, roundrobin, sticky, cooperative sticky 전략은 이동량, 균등성, rebalance 중단 범위가 다르다.

## 예시

```text
partition.assignment.strategy=CooperativeStickyAssignor
watch: moved_partitions, rebalance_duration, assignment_skew
```

전략 선택은 partition 수를 나누는 문제가 아니라 배포 중 처리 중단과 state 이동 비용을 줄이는 문제다.

## 면접 답변 예시

> Kafka partition assignment strategy는 group의 partition을 consumer에 어떻게 나누고 rebalance 때 얼마나 이동할지 정합니다. Range는 topic별 분포가 치우칠 수 있고 sticky는 기존 assignment를 최대한 유지하며 cooperative sticky는 한 번에 전부 revoke하는 중단을 줄입니다. Cooperative 방식으로 바꿀 때는 client version과 assignor 목록의 호환성을 확인하고 rolling migration 절차를 따라야 합니다. 변경 전후 moved partition, rebalance duration과 lag를 비교해 state 이동 비용이 실제로 줄었는지 보겠습니다.

## 장점

- sticky 전략은 stateful consumer의 cache 이동을 줄인다.

## 단점

- partition 수가 consumer 수보다 적으면 균등성이 제한된다.

## 주의사항 / 실무 팁

- consumer client 버전이 cooperative assignor를 지원하는지 확인한다.
