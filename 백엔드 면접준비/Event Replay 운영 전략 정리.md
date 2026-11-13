# Event Replay 운영 전략 정리

## 핵심 요약

- 손상된 projection을 원본 log에서 재구성한다.
- email·payment 같은 side effect가 다시 실행될 수 있다.
- replay mode에서 외부 side effect를 강제 차단한다.

## 개념 설명

event replay는 과거 log의 지정 offset부터 event를 다시 소비해 projection을 복구하거나 새 handler 결과를 만드는 운영 작업이다.

replay 전용 consumer group과 target store를 사용하고 checkpoint, 처리 속도, side effect 차단 정책을 분리해 live traffic과 격리한다.

## 예시

```text
source topic=orders offset 0..8,400,000
replay group=projection-v3-rebuild
side effect=email/payment disabled
target=order_summary_v3
```

과거 event를 현재 handler로 실행할 때 schema와 업무 의미가 달라질 수 있으므로 versioned decoder와 dry-run count가 필요하다.

## 면접 답변 예시

> Event replay는 과거 event를 다시 소비해 손상된 projection을 복구하거나 새 read model을 만드는 작업입니다. Live consumer와 다른 group·target store를 사용하고 replay context에서는 email이나 payment 같은 외부 side effect를 코드 수준에서 차단하겠습니다. 과거 schema와 현재 업무 의미가 다를 수 있으므로 versioned decoder, dry-run count와 checksum으로 결과를 먼저 비교합니다. 처리 rate와 pause switch를 두고 offset별 실패·skip·checkpoint를 감사 기록으로 남긴 뒤 검증된 결과만 전환합니다.

## 장점

- 새 read model을 live 전환 전에 검증한다.

## 단점

- replay traffic이 broker와 DB를 포화시킨다.

## 주의사항 / 실무 팁

- 처리 rate limit과 pause switch를 둔다.
