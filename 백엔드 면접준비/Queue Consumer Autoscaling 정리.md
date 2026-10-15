# Queue Consumer Autoscaling 정리

## 핵심 요약

- 트래픽 증가 때 소비 지연을 자동으로 줄일 수 있다.
- downstream 용량을 넘겨 scale out하면 DB나 API가 먼저 무너진다.
- scale 기준에 lag와 oldest age를 함께 넣는다.

## 개념 설명

Queue Consumer Autoscaling은 큐 적체와 처리량을 보고 consumer 수를 자동 조절하는 운영 방식이다.

lag, oldest message age, consume rate, per-consumer 처리 시간으로 필요한 replica 수를 계산하되, downstream 용량과 max concurrency를 함께 제한한다.

## 예시

```text
scale out if oldest_message_age > 2m and cpu < 70%
max consumers = db_pool_size / worker_db_conn
scale in after lag stable for 15m
```

consumer만 늘리면 해결된다는 보장은 없다. DB나 외부 API가 병목이면 autoscaling이 장애를 키운다.

## 면접 답변 예시

> Queue consumer autoscaling은 CPU만 보지 않고 lag, oldest message age와 처리율로 필요한 consumer 수를 조절하는 방식입니다. Oldest age는 사용자가 기다린 시간에 가깝지만 poison message나 retry loop 하나가 값을 왜곡할 수 있어 정상 backlog와 DLQ 흐름을 분리하겠습니다. Consumer를 늘려도 DB pool이나 외부 API quota가 병목이면 장애가 커지므로 downstream capacity로 max replica를 제한해야 합니다. Scale-in에는 안정 시간을 두어 rebalance와 cold cache가 반복되지 않게 하고 실제 drain 속도를 확인합니다.

## 장점

- 야간이나 저부하 시간에는 비용을 낮출 수 있다.

## 단점

- 빠른 scale in은 rebalance와 cache cold start를 반복시킨다.

## 주의사항 / 실무 팁

- DB connection과 외부 API quota로 max replica를 제한한다.
