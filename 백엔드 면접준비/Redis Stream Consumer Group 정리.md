# Redis Stream Consumer Group 정리

## 핵심 요약

- Redis만으로 가벼운 작업 큐를 만들 수 있다.
- stream trimming을 잘못하면 아직 처리하지 않은 entry를 잃을 수 있다.
- XPENDING age와 pending count를 알림에 둔다.

## 개념 설명

Redis Stream Consumer Group은 여러 consumer가 stream entry를 나눠 읽고 ack로 완료를 표시하는 Redis Streams 기능이다.

XREADGROUP으로 새 메시지를 받고, 처리 후 XACK한다. 장애 consumer가 남긴 pending entry는 XPENDING과 XCLAIM으로 회수한다.

## 예시

```text
XGROUP CREATE orders g1 $ MKSTREAM
XREADGROUP GROUP g1 c1 COUNT 10 STREAMS orders >
XACK orders g1 1690000000-0
XPENDING orders g1
```

pending entry는 처리 중이지만 완료되지 않은 메시지다. ack 누락과 consumer 장애를 회수 정책으로 다뤄야 한다.

## 면접 답변 예시

> Redis Stream consumer group은 여러 consumer가 entry를 나눠 읽고 처리 후 `XACK`으로 완료를 표시하는 방식입니다. Consumer가 죽으면 message가 pending list에 남으므로 idle time과 delivery count를 확인한 뒤 다른 consumer가 claim할 수 있습니다. Claim은 중복 처리를 만들 수 있어 handler는 멱등해야 하고, 기준 시간은 정상 처리 timeout보다 길게 잡겠습니다. Stream trimming이 아직 읽지 않았거나 pending인 entry를 제거하지 않도록 lag와 보관 정책을 함께 관리합니다.

## 장점

- consumer별 pending 상태를 추적할 수 있다.

## 단점

- pending이 쌓이면 메모리와 재처리 부담이 커진다.

## 주의사항 / 실무 팁

- claim 기준 시간을 처리 timeout보다 길게 잡는다.
