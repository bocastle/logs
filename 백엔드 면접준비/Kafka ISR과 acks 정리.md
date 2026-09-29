# Kafka ISR과 acks 정리

## 핵심 요약

- broker 장애 중에도 기록 손실 위험을 줄일 수 있다.
- min.insync.replicas를 높이면 broker 장애 때 produce 실패가 늘 수 있다.
- acks와 min.insync.replicas를 함께 설정한다.

## 개념 설명

Kafka ISR과 acks는 producer가 record를 성공으로 볼 때 leader와 in-sync replica 중 어디까지 복제를 확인할지 정하는 내구성 조건이다.

`acks=all`은 ISR의 충분한 replica가 record를 받은 뒤 성공을 응답한다. min.insync.replicas와 함께 봐야 실제 손실 허용 범위가 정해진다.

## 예시

```text
replication.factor=3
min.insync.replicas=2
producer acks=all
ISR=[broker-1, broker-2]
```

`acks=all`만 적으면 충분하지 않다. ISR이 줄었을 때 write를 거절할지, 가용성을 택할지 정책이 함께 필요하다.

## 면접 답변 예시

> Kafka에서 `acks=all`은 leader가 현재 ISR replica들의 기록 확인을 받은 뒤 producer에 성공을 응답하게 합니다. 하지만 ISR이 1개까지 줄어도 write를 허용하면 내구성이 약해지므로 `min.insync.replicas`를 함께 설정해야 합니다. Replication factor 3에 min ISR 2라면 ISR이 2개 미만일 때 availability보다 내구성을 택해 produce를 실패시킵니다. 중요 topic별로 정책을 정하고 under-replicated partition, ISR shrink와 produce error를 함께 관찰하겠습니다.

## 장점

- producer 성공 기준을 복제 상태와 연결해 설명할 수 있다.

## 단점

- ISR 축소를 방치하면 acks=all의 의미가 약해진다.

## 주의사항 / 실무 팁

- under replicated partitions와 ISR shrink count를 알림으로 둔다.
