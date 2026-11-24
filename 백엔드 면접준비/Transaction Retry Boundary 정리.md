# Transaction Retry Boundary 정리

## 핵심 요약

- 동시성 오류를 새 snapshot에서 다시 시도한다.
- 외부 side effect가 boundary 안에 있으면 중복된다.
- transaction proxy 바깥에 retry를 둔다.

## 개념 설명

transaction retry boundary는 serialization failure나 deadlock처럼 rollback된 DB 작업을 어디부터 다시 실행할지 정하는 범위다.

new transaction을 만드는 service 바깥에서 전체 unit of work를 재호출하고 내부 변경은 idempotency key 또는 unique constraint로 중복 안전하게 만든다.

## 예시

```text
retry boundary { begin tx -> read -> update -> commit }
serialization failure -> rollback entire tx -> jitter -> new tx
external payment call은 boundary 밖 또는 idempotency 보호
```

rollback-only transaction 내부에서 같은 repository call만 다시 실행하면 이미 실패 상태인 connection을 재사용하고 snapshot도 새로 얻지 못한다.

## 면접 답변 예시

> Transaction retry boundary는 deadlock이나 serialization failure로 rollback된 unit of work를 어디부터 새 transaction으로 다시 실행할지 정하는 범위입니다. 실패한 transaction 안의 repository call 하나만 반복하지 않고 transaction proxy 바깥에서 read, 판단과 write 전체를 새 snapshot으로 재실행하겠습니다. SQLSTATE allowlist, 최대 횟수와 전체 deadline을 두고 backoff·jitter로 경합을 줄입니다. Payment나 message 발행 같은 외부 side effect는 boundary 밖으로 옮기거나 outbox와 idempotency key로 중복을 막은 뒤 최종 오류 원인을 보존합니다.

## 장점

- transaction 전체 불변식을 보존한다.

## 단점

- 무제한 retry는 DB 경합을 증폭한다.

## 주의사항 / 실무 팁

- SQLSTATE allowlist와 최대 횟수를 설정한다.
