# Kafka Poison Message 격리 정리

## 핵심 요약

- 단일 나쁜 메시지가 partition 처리를 막는 것을 줄인다.
- 격리 기준이 느슨하면 일시 장애 메시지를 너무 빨리 버린다.
- 오류 유형별 retryable 여부와 최대 횟수를 정한다.

## 개념 설명

Kafka Poison Message 격리는 특정 메시지가 계속 consumer 실패를 유발할 때 주 처리 흐름을 막지 않도록 별도 topic이나 저장소로 분리하는 절차다.

재시도 횟수와 오류 유형을 보고 더 이상 진행할 수 없는 record를 DLQ나 quarantine topic으로 보내고 원본 offset은 커밋한다.

## 예시

```text
consume record offset=881
validation error repeats 5 times
Kafka transaction:
  publish to orders.poison with reason
  commit source offset 882
```

poison message를 계속 붙잡으면 같은 partition 전체가 멈춘다. 격리 후 원인 분석과 replay 조건을 따로 둬야 한다.

## 면접 답변 예시

> Poison message는 같은 record가 반복 실패해 partition 뒤의 정상 message까지 막는 상황입니다. Validation처럼 영구 오류와 timeout 같은 일시 오류를 나눠 제한 retry 후 quarantine으로 보내겠습니다. Kafka 안에서는 quarantine record 발행과 source offset commit을 transaction으로 묶어 둘 사이 crash의 누락·중복을 줄이고, transaction을 쓸 수 없다면 source identity로 격리 write를 멱등 처리합니다. Payload의 개인정보를 최소화하고 owner, 수정·replay 승인과 처리 기한이 있는 운영 흐름을 둡니다.

## 장점

- 오류 payload와 reason을 남겨 원인 분석이 쉬워진다.

## 단점

- DLQ 운영자가 없으면 격리 메시지가 영구 방치된다.

## 주의사항 / 실무 팁

- poison topic에는 reason, stack class, source offset을 남긴다.
