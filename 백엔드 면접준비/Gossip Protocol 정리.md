# Gossip Protocol 정리

## 핵심 요약

- 중앙 coordinator 없이 membership을 확산한다.
- 수렴 전 node마다 topology가 다를 수 있다.
- payload에는 version과 incarnation을 둔다.

## 개념 설명

gossip protocol은 각 node가 무작위 peer와 membership과 상태 version을 주기적으로 교환해 cluster 전체로 확산하는 분산 통신 방식이다.

heartbeat incarnation과 failure detector suspicion을 섞어 장애 정보를 전파하며 fan-out과 interval이 수렴 시간과 network 비용을 결정한다.

## 예시

```text
A gossips {B:alive v12, C:suspect v8} to D
D merges higher versions and forwards in next round
```

failure detector의 suspect는 확정 사망이 아니므로 일시 network partition에서 즉시 replica를 제거하면 불필요한 재배치가 생긴다.

## 면접 답변 예시

> Gossip protocol은 각 node가 무작위 peer와 membership·상태 version을 주기적으로 교환해 중앙 coordinator 없이 정보를 퍼뜨리는 방식입니다. Version과 incarnation으로 중복·역순 message를 merge할 수 있지만 수렴 전에는 node마다 topology view가 다를 수 있습니다. Failure detector의 suspect를 즉시 dead로 확정하면 network jitter에 불필요한 재배치가 생겨 grace period와 확인 절차를 둬야 합니다. Fan-out, interval과 payload 크기를 조정하며 convergence time, traffic과 false positive를 함께 측정합니다.

## 장점

- node 수가 늘어도 fan-out을 제한한다.

## 단점

- 느린 node를 장애로 오인할 수 있다.

## 주의사항 / 실무 팁

- suspect 후 confirm까지 grace period를 둔다.
