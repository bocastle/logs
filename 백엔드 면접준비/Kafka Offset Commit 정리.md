# Kafka Offset Commit 정리

## 핵심 요약

- Kafka offset commit은 consumer group이 partition별로 다음에 읽을 위치를 broker에 저장하는 과정이다.
- 처리 전에 commit하면 장애 시 메시지를 건너뛰고, 처리 뒤 commit 실패는 같은 메시지를 다시 처리하게 한다.
- 외부 side effect를 멱등하게 만들고 batch 크기·commit 주기로 허용할 재처리 범위를 정한다.

## 개념 설명

Consumer가 `poll()`로 받은 record를 처리한 뒤 어디까지 끝냈는지 저장하지 않으면 재시작 때 위치를 알 수 없다. Committed offset은 마지막 처리 record가 아니라 다음에 읽을 offset이다.

처리 성공과 commit의 원자성은 자동으로 생기지 않는다. 먼저 commit하면 처리 전 장애에서 손실이 생기고, 처리 후 commit하면 commit 실패 때 중복이 생긴다. 일반적으로 at-least-once를 선택하고 consumer side effect를 event ID로 멱등하게 만든다.

## 예시

```text
poll records p0:100..120
process ok
commitSync({ p0: offset 121 })
restart -> resumes from 121
```

`100..120` 처리가 끝났다면 121을 commit한다. 여러 partition을 batch로 처리할 때는 성공한 partition의 위치만 저장하고 revoke 전에 진행 상태를 정리하는 정책이 필요하다.

## 면접 답변 예시

> Kafka offset commit은 consumer group이 partition에서 다음에 읽을 위치를 저장하는 과정입니다. 처리 전에 commit하면 장애 시 메시지를 잃을 수 있어 보통 처리가 끝난 뒤 다음 offset을 commit합니다. 이 경우 commit 실패로 중복 처리가 가능하므로 consumer의 DB 쓰기와 외부 호출을 멱등하게 만듭니다. Batch 크기와 commit 주기로 재처리 범위를 조절하고 commit latency·failure와 lag를 함께 관찰합니다.

## 장점

- 수동 commit으로 업무 처리 완료 시점과 offset 저장 시점을 맞출 수 있다.
- Group 재시작 후 partition별 진행 위치에서 이어 읽을 수 있다.

## 단점

- 처리 후 commit 실패는 같은 record의 재처리와 중복 side effect를 만든다.
- Batch와 commit 간격이 크면 장애 시 다시 처리할 범위가 넓어진다.

## 주의사항 / 실무 팁

- Event ID와 business key를 사용한 idempotent consumer로 중복 처리를 견딘다.
- Rebalance revoke, 부분 batch 실패와 commit 실패를 포함한 통합 테스트를 둔다.
