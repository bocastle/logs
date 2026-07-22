# Kafka consumer lag 알림 임계값 잡는 법 정리

## 핵심 요약

- Kafka lag의 메시지 수만으로는 사용자 지연을 알기 어려워 oldest message age와 생산·소비 속도를 함께 본다.
- Topic과 consumer group의 신선도 SLO에 맞춰 warning·critical 지속 시간을 다르게 둔다.
- 배포 중 rebalance와 partition skew를 구분해 짧은 정상 spike가 바로 장애 호출로 이어지지 않게 한다.

## 개념 설명

Consumer lag은 partition의 최신 offset과 consumer group이 commit한 offset의 차이다. 같은 lag 50,000도 초당 수만 건을 처리하는 topic과 하루 수천 건인 topic에서 의미가 전혀 다르다.

사용자 영향과 가까운 지표는 가장 오래 기다린 메시지의 age다. Lag 증가 속도와 produce·consume rate를 함께 보면 일시적인 backlog인지 영구적으로 뒤처지는 상태인지 구분할 수 있다. 평균만 보면 한 partition의 skew를 숨길 수 있어 partition별 최댓값도 확인한다.

## 예시

```text
group=order-projector
lag > 50_000 for 10m
or oldest_message_age > 5m
and consume_rate < produce_rate
```

조건은 예시이며 업무 SLO에 맞춰야 한다. 배포와 rebalance 직후의 짧은 spike는 지속 시간 조건으로 흡수하고, consume rate가 회복되지 않는 경우에 더 높은 심각도를 준다.

## 면접 답변 예시

> Kafka lag 알림은 메시지 개수만 고정하지 않고 oldest message age, lag 증가 속도와 produce·consume rate를 함께 사용하겠습니다. 임계값은 주문 처리와 분석 pipeline처럼 업무 신선도 SLO가 다른 group마다 따로 둡니다. Partition 평균이 아니라 최댓값도 봐 skew와 느린 consumer를 찾습니다. 배포 중 rebalance spike는 지속 시간으로 걸러 내고 처리 오류율과 DLQ 증가도 같은 알림 맥락에 포함합니다.

## 장점

- Partition별 지연을 보면 느린 consumer와 key skew를 구분하기 쉽다.
- Oldest message age를 SLO에 연결해 사용자 영향 전 대응할 수 있다.

## 단점

- Lag만 보면 처리 실패를 commit해 backlog가 없는 것처럼 보이는 상황을 놓칠 수 있다.
- Rebalance와 일시적인 produce burst를 고려하지 않으면 배포 때마다 오탐이 발생한다.

## 주의사항 / 실무 팁

- 배포 시간대의 rebalance count와 partition별 lag 최댓값을 함께 본다.
- 알림에 처리 성공률, DLQ 유입과 최근 배포 정보를 붙여 바로 원인을 좁힐 수 있게 한다.
