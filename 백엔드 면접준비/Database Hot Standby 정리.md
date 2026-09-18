# Database Hot Standby 정리

## 핵심 요약

- standby를 실시간 read replica로 활용한다.
- 긴 read query가 recovery conflict로 취소된다.
- recovery conflict와 replay lag를 함께 본다.

## 개념 설명

hot standby는 PostgreSQL standby가 WAL recovery를 계속 수행하면서 read-only query를 허용하는 복제 상태다.

replay가 row version을 제거하거나 lock을 재현해야 할 때 standby query와 recovery conflict가 생기며 max_standby_streaming_delay 뒤 query를 취소할 수 있다.

## 예시

```text
standby: read-only SELECT 실행
primary: VACUUM이 old tuple 제거
replay: recovery conflict -> canceling statement
```

hot_standby_feedback는 일부 취소를 줄이지만 primary의 dead tuple 정리를 늦춰 bloat를 키울 수 있다.

## 면접 답변 예시

> PostgreSQL hot standby는 WAL을 replay하는 standby에서 read-only query도 실행할 수 있게 하는 상태입니다. Read 부하를 primary에서 분리하고 승격 후보를 유지할 수 있지만 replay와 query가 충돌하면 설정된 지연 뒤 query가 취소될 수 있습니다. `hot_standby_feedback`로 일부 취소를 줄이면 primary가 dead tuple을 오래 보존해 bloat가 커질 수 있어 recovery conflict, replay lag와 dead tuple age를 함께 보겠습니다. 긴 분석 query는 별도 replica에 두고 standby와 독립적인 backup도 따로 유지합니다.

## 장점

- read-only 조회를 primary에서 분리하고 승격 가능한 replica를 유지한다.

## 단점

- replay 지연은 stale read를 늘린다.

## 주의사항 / 실무 팁

- 분석 query에는 별도 replica를 고려한다.
- standby는 독립 backup을 대체하지 않는다.
