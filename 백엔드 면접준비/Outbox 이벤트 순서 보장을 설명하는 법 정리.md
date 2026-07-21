# Outbox 이벤트 순서 보장을 설명하는 법 정리

## 핵심 요약

- Outbox는 업무 데이터와 이벤트 기록을 같은 DB 트랜잭션에 저장해 commit과 발행 사이의 유실을 막는다.
- 순서 보장은 전역이 아니라 보통 aggregate 단위이며 증가 version과 같은 broker partition key가 필요하다.
- Relay 재시도로 중복 발행될 수 있으므로 consumer 멱등성은 별도로 유지한다.

## 개념 설명

Outbox pattern은 주문 변경과 발행할 이벤트 행을 같은 데이터베이스 트랜잭션에 저장한다. Relay가 commit된 outbox를 읽어 broker에 보내므로 DB 변경은 성공했지만 이벤트가 유실되는 구간을 줄인다.

같은 주문의 이벤트 순서가 중요하다면 `aggregate_id`와 단조 증가 version에 unique 제약을 둔다. Relay는 version 순으로 읽고 `aggregate_id`를 partition key로 사용해 같은 aggregate가 같은 partition에 가게 한다. 여러 relay가 병렬 처리할 때도 같은 aggregate를 동시에 가져가지 않게 해야 한다.

## 예시

```sql
INSERT INTO outbox_events(aggregate_id, version, event_type, payload)
VALUES (:order_id, :version, 'OrderPaid', :payload);
-- UNIQUE (aggregate_id, version)
```

서로 다른 주문 사이의 전역 순서는 보장하지 않는다. Broker 전송 성공 뒤 outbox 상태 갱신 전에 relay가 죽으면 같은 이벤트를 다시 보낼 수 있으므로 event ID 기반 중복 제거가 필요하다.

## 면접 답변 예시

> Outbox는 업무 데이터와 이벤트를 같은 DB 트랜잭션에 저장해 commit과 발행 사이의 유실을 막는 패턴입니다. 순서가 필요한 aggregate에는 증가 version과 unique 제약을 두고 relay가 같은 aggregate ID를 broker partition key로 사용하게 합니다. 이 보장은 aggregate 내부 순서이지 전체 이벤트의 전역 순서는 아닙니다. Relay 재시도로 중복 발행될 수 있으므로 consumer는 event ID로 멱등하게 처리하고 outbox 정리 지연도 관찰합니다.

## 장점

- Aggregate 단위 ordering key와 version을 데이터 모델에 명시할 수 있다.
- DB commit과 broker 발행 사이의 이벤트 유실을 복구 가능한 행으로 남긴다.

## 단점

- 발행 성공 뒤 상태 갱신 전에 relay가 실패하면 중복 이벤트가 생긴다.
- Outbox가 계속 쌓이면 polling index, table bloat와 vacuum 비용이 커진다.

## 주의사항 / 실무 팁

- Broker partition key를 `aggregate_id`로 고정하고 relay 병렬화가 같은 aggregate를 나누지 않게 한다.
- Event ID 중복 처리와 outbox backlog age, relay 재시도 수를 함께 관찰한다.
