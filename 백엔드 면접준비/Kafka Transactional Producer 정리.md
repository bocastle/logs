# Kafka Transactional Producer 정리

## 핵심 요약

- Kafka Transactional Producer는 여러 레코드 전송과 consumer offset 갱신을 하나의 Kafka 트랜잭션으로 묶는다.
- 핵심은 `transactional.id`를 안정적으로 관리하고 소비자가 `read_committed`로 완료된 레코드만 읽게 하는 것이다.
- Kafka 내부 작업은 원자적으로 묶을 수 있지만 외부 데이터베이스까지 자동으로 같은 트랜잭션이 되는 것은 아니다.

## 개념 설명

프로듀서가 `initTransactions`로 coordinator와 epoch를 확정한 뒤 `beginTransaction`, 전송, `commitTransaction` 순서로 처리한다. 같은 `transactional.id`를 가진 오래된 프로듀서는 fencing되어 중복 writer가 되는 것을 막는다.

consume-transform-produce 흐름에서는 처리한 offset을 `sendOffsetsToTransaction`으로 함께 커밋해야 입력 소비와 출력 발행이 원자적 경계에 들어간다.

## 예시

```text
transactional.id=order-enricher-3
enable.idempotence=true

beginTransaction()
send(enriched-order)
sendOffsetsToTransaction(consumedOffsets, groupMetadata)
commitTransaction()

consumer isolation.level=read_committed
```

출력 레코드와 입력 offset을 같이 커밋하므로 재시작 후 같은 입력을 다시 처리해도 완료되지 않은 결과는 소비자에게 보이지 않는다.

## 면접 답변 예시

> Kafka Transactional Producer는 여러 record 전송과 consumer offset commit을 하나의 Kafka transaction으로 묶는 기능입니다. Instance마다 안정적이고 충돌하지 않는 `transactional.id`를 사용해 오래된 producer를 fencing하고, consumer는 `read_committed`로 완료된 결과만 읽어야 합니다. Consume-transform-produce 흐름에서는 offset도 `sendOffsetsToTransaction()`으로 같이 넣습니다. 이 원자성은 Kafka 밖의 DB와 HTTP 호출에는 적용되지 않으므로 outbox나 idempotency key를 별도로 설계합니다.

## 장점

- Kafka 안에서 consume-process-produce 흐름의 중복 노출을 줄인다.
- offset과 결과 레코드의 커밋 순서를 애플리케이션이 따로 맞출 필요가 줄어든다.
- producer fencing으로 오래된 인스턴스의 쓰기를 차단할 수 있다.

## 단점

- transaction coordinator와 commit 과정 때문에 지연과 운영 복잡도가 늘어난다.
- 긴 트랜잭션은 timeout과 aborted record 누적 위험을 키운다.
- 외부 DB나 HTTP 호출은 Kafka 트랜잭션에 포함되지 않는다.

## 주의사항 / 실무 팁

- `transactional.id`가 재배포 후에도 충돌 없이 안정적으로 유지되는지 확인한다.
- transaction abort rate, commit latency, coordinator 오류를 함께 관찰한다.
- 외부 부작용에는 idempotency key나 outbox를 적용한다.
