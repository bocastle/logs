# Redis Pub-Sub와 Redis Streams 비교 정리

## 핵심 요약

- Pub-Sub은 낮은 지연으로 실시간 알림을 보내기 쉽다.
- Pub-Sub은 구독자가 끊겨 있으면 메시지를 잃는다.
- 손실 허용 여부를 먼저 결정한다.

## 개념 설명

Redis Pub-Sub와 Redis Streams 비교는 즉시 fan-out 알림과 보관 가능한 메시지 스트림 중 어느 모델이 필요한지 고르는 문제다.

Pub-Sub은 구독 중인 client에게만 메시지를 보내고 저장하지 않는다. Streams는 entry id, consumer group, pending list로 재처리와 ack를 지원한다.

## 예시

```text
Pub-Sub: PUBLISH news "x" -> online subscribers only
Streams: XADD orders * type paid
XREADGROUP GROUP g1 c1 STREAMS orders >
XACK orders g1 169...
```

알림 손실이 허용되면 Pub-Sub이 단순하고, 처리 보장과 재시도가 필요하면 Streams가 맞다.

## 면접 답변 예시

> Redis Pub-Sub은 현재 연결된 subscriber에게 즉시 fan-out하지만 message를 저장하지 않아 끊긴 동안의 알림은 받을 수 없습니다. Redis Streams는 entry ID와 consumer group, pending entry list가 있어 ack되지 않은 작업을 확인하고 다시 전달할 수 있습니다. 다만 Streams도 자동으로 exactly-once를 보장하는 것은 아니므로 consumer를 멱등하게 만들고 pending 회수와 trimming 정책을 운영해야 합니다. 손실 가능한 실시간 신호에는 Pub-Sub, 재처리와 처리 추적이 필요한 작업에는 Streams를 선택하겠습니다.

## 장점

- Streams는 consumer group과 ack로 작업 큐처럼 운영할 수 있다.

## 단점

- Streams는 trimming과 pending 관리가 필요하다.

## 주의사항 / 실무 팁

- Streams는 XTRIM과 pending entry 회수 정책을 둔다.
