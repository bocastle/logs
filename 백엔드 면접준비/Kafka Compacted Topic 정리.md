# Kafka Compacted Topic 정리

## 핵심 요약

- consumer가 최신 상태를 topic에서 재구성하기 좋다.
- 전체 이벤트 이력 감사에는 적합하지 않을 수 있다.
- 상태 topic과 감사 event topic을 분리한다.

## 개념 설명

Kafka Compacted Topic은 같은 key의 여러 record 중 최신 값을 보존하도록 log compaction을 적용한 topic이다.

broker cleaner가 segment를 훑으며 key별 최신 record와 tombstone 보존 기간을 남긴다. 상태 재구성용 changelog에 잘 맞는다.

## 예시

```text
key=user-7 value=v1
key=user-7 value=v2
key=user-7 value=null  # tombstone
compaction -> latest per key remains until delete retention
```

compaction은 즉시 삭제가 아니라 비동기 정리다. consumer는 중간 record를 한동안 볼 수 있고, 순수 이벤트 히스토리 보존과는 목적이 다르다.

## 면접 답변 예시

> Compacted topic은 같은 key의 record 중 최신 상태를 장기적으로 남기도록 log cleaner가 비동기로 정리하는 topic입니다. 정리 전에는 이전 record도 보일 수 있어 즉시 중복이 사라진다고 가정하면 안 됩니다. 삭제는 tombstone을 발행하고 보존 기간 동안 consumer에 전달될 시간을 줍니다. 상태 changelog에는 적합하지만 전체 감사 이력이 필요하면 별도 append-only event topic을 유지하겠습니다.

## 장점

- key별 오래된 값을 줄여 저장 비용을 낮춘다.

## 단점

- tombstone 보존 기간 설정을 잘못하면 삭제 전파가 흔들린다.

## 주의사항 / 실무 팁

- tombstone과 delete.retention.ms 정책을 테스트한다.
