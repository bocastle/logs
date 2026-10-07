# Kafka Exactly-once 처리 한계 정리

## 핵심 요약

- Kafka의 exactly-once는 Kafka 안의 레코드 발행과 consumer offset 커밋을 한 트랜잭션으로 묶는 보장이다.
- 소비자가 `read_committed`를 사용해야 중단되거나 abort된 트랜잭션의 레코드를 건너뛸 수 있다.
- 외부 DB, HTTP 호출, 이메일 전송까지 자동으로 exactly-once가 되지는 않으므로 별도의 멱등 설계가 필요하다.

## 개념 설명

consume-transform-produce 흐름에서는 결과 레코드와 입력 offset을 같은 Kafka 트랜잭션으로 커밋해 재처리 시 중간 결과가 노출되지 않게 한다.

보장 범위는 Kafka transaction log 안쪽이다. 외부 시스템의 부작용은 commit 결과를 알기 전에 성공할 수 있어 outbox, idempotency key, 중복 제거가 필요하다.

## 예시

```text
Kafka transaction:
  produce(output) + commit(input offset) -> atomic

Outside Kafka:
  update external DB -> timeout before response
  retry -> same update may run again

consumer isolation.level=read_committed
```

Kafka 결과 발행과 offset은 한 번처럼 보이지만 외부 DB 갱신은 별도 경계다. 재시도해도 결과가 같도록 business key와 멱등 기록을 둬야 한다.

## 면접 답변 예시

> Kafka exactly-once는 consume-transform-produce 흐름에서 결과 record와 입력 offset commit을 하나의 Kafka transaction으로 묶는 보장입니다. Consumer가 `read_committed`를 사용해야 abort된 transaction의 결과가 보이지 않습니다. 이 범위는 Kafka transaction log 안쪽이라 외부 DB update, HTTP 호출과 email 발송까지 한 번만 실행되게 하지는 않습니다. 외부 부작용에는 outbox, business id 기반 idempotency key와 중복 처리 이력을 별도로 적용하겠습니다.

## 장점

- Kafka 내부 재처리에서 중간 결과와 중복 노출을 줄인다.
- 결과 레코드와 입력 offset의 커밋 순서를 하나의 경계로 관리한다.
- abort된 트랜잭션을 소비자에게 숨길 수 있다.

## 단점

- 외부 DB나 API 호출에는 exactly-once가 자동으로 이어지지 않는다.
- transaction coordinator와 abort 처리로 지연과 운영 복잡도가 늘어난다.
- 긴 트랜잭션은 timeout과 처리 정체를 키울 수 있다.

## 주의사항 / 실무 팁

- consumer의 `isolation.level=read_committed`를 확인한다.
- 외부 부작용은 business id 기반 멱등 키와 처리 이력으로 보호한다.
- abort rate, commit latency, 재처리 건수를 함께 관찰한다.
