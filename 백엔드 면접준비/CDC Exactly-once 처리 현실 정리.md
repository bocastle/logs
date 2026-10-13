# CDC Exactly-once 처리 현실 정리

## 핵심 요약

- 재시작과 redelivery 뒤 같은 sink 상태로 수렴한다.
- 비멱등 외부 호출은 중복 부작용을 낸다.
- sink는 natural key 기반 idempotent upsert를 쓴다.

## 개념 설명

CDC exactly-once는 source DB부터 외부 sink까지 한 번만 처리된다는 절대 보장보다 중복 전달을 idempotent upsert로 같은 결과에 수렴시키는 운영 목표다.

WAL LSN과 table primary key를 event identity로 사용하고 sink mutation과 offset checkpoint를 같은 transaction에 저장할 수 있을 때 처리 원자성을 근사한다.

## 예시

```text
event_id=(orders,42,lsn=16/B380120)
sink: idempotent upsert order 42
checkpoint advances to 16/B380120 in same transaction
```

source offset commit과 외부 API side effect를 하나의 transaction으로 묶을 수 없으면 crash 구간에서 duplicate가 반드시 가능하다.

## 면접 답변 예시

> CDC의 exactly-once는 source부터 모든 외부 sink까지 절대 한 번만 실행된다는 뜻보다 redelivery가 와도 같은 sink 상태로 수렴시키는 목표로 보는 편이 현실적입니다. Connector의 안정적인 event position과 business key로 mutation을 멱등 upsert하고, 가능한 sink에서는 mutation과 checkpoint를 같은 transaction에 저장하겠습니다. 외부 API처럼 offset과 묶을 수 없는 부작용에는 duplicate crash window가 남으므로 idempotency key가 필요합니다. 이름만 믿지 않고 write 전·후와 checkpoint 전·후 crash를 재현하며 slot lag와 WAL 보관량도 함께 관찰합니다.

## 장점

- LSN 기반으로 누락과 중복 구간을 추적한다.

## 단점

- schema 변경이 event identity와 decoder를 깨뜨릴 수 있다.

## 주의사항 / 실무 팁

- event id와 offset을 감사 로그에 남긴다.
