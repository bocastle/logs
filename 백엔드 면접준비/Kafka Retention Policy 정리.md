# Kafka Retention Policy 정리

## 핵심 요약

- 장애 복구와 consumer 재처리 가능 기간을 명확히 정할 수 있다.
- retention이 짧으면 느린 consumer가 offset out of range를 만난다.
- consumer 최대 중단 시간을 기준으로 replay window를 정한다.

## 개념 설명

Kafka Retention Policy는 topic의 record를 시간이나 크기 기준으로 얼마나 보관할지 정하는 정책이다.

`retention.ms`, partition별 `retention.bytes`, `segment.ms` 설정에 따라 오래된 segment가 삭제된다. replay 가능 기간과 저장 비용 사이의 계약이다.

## 예시

```text
topic=orders.events
retention.ms=604800000
retention.bytes=500GB
watch: log_size, oldest_offset_age, replay_window
```

retention은 백업이 아니다. 기간이 지나면 consumer가 필요한 offset을 잃고 replay가 불가능해질 수 있다.

## 면접 답변 예시

> Kafka retention은 record를 얼마나 오래 replay할 수 있는지와 broker 저장 비용을 정하는 정책입니다. Consumer의 최대 중단 시간보다 replay window가 짧으면 복구 시 필요한 offset이 이미 삭제될 수 있습니다. `retention.bytes`는 partition별 기준이므로 partition 수와 replication factor까지 포함해 disk 용량을 계산하고, segment 단위 삭제라 정확히 해당 byte와 시각에서 잘리는 것은 아니라는 점도 고려하겠습니다. Compaction과 delete 정책, 법적 감사 보관은 목적이 다르므로 장기 보관이 필요하면 외부 archival을 별도로 설계합니다.

## 장점

- topic별 저장 비용을 제한할 수 있다.

## 단점

- 길게 잡으면 broker disk와 복제 비용이 커진다.

## 주의사항 / 실무 팁

- broker disk usage와 oldest offset age를 알림에 둔다.
- 전체 저장량을 추정할 때 partition 수와 replication factor를 반영한다.
