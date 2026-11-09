# Queue Delay와 Throughput 정리

## 핵심 요약

- 소비자 증설이나 batch 조정 필요성을 정량화할 수 있다.
- 평균 throughput만 보면 특정 partition 지연을 놓칠 수 있다.
- produce rate, consume rate, oldest age를 한 화면에 둔다.

## 개념 설명

Queue Delay와 Throughput은 큐에 들어온 메시지가 처리되기까지의 대기 시간과 단위 시간당 처리량을 함께 보는 성능 관점이다.

생산 속도보다 소비 throughput이 낮으면 lag와 oldest message age가 늘어난다. 평균 처리량보다 burst와 tail delay를 봐야 한다.

## 예시

```text
produce_rate=5k/min
consume_rate=3k/min
oldest_message_age=12m
needed consumers = produce_rate / per_consumer_rate
```

throughput이 높아도 오래된 메시지가 남으면 사용자 신선도는 나쁘다. delay와 처리량을 같이 봐야 한다.

## 면접 답변 예시

> Queue 운영에서는 초당 처리량과 message가 기다린 delay를 같이 봐야 합니다. Consume rate가 높아도 특정 partition의 오래된 message가 남으면 사용자 freshness는 나쁠 수 있어 produce·consume rate, lag와 oldest age를 partition별로 확인하겠습니다. Consumer 증설 전에는 DB나 외부 API가 병목인지 확인해 scale-out이 downstream을 더 압박하지 않게 합니다. Batch size를 키우면 throughput은 늘어도 개별 delay가 길어질 수 있어 p95 age와 retry·poison message를 함께 비교합니다.

## 장점

- 사용자 영향에 가까운 oldest age 기준을 만들 수 있다.

## 단점

- batch를 키우면 throughput은 늘어도 개별 delay가 길어질 수 있다.

## 주의사항 / 실무 팁

- partition별 delay를 별도로 확인한다.
