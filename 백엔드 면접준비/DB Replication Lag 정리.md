# DB Replication Lag 정리

## 핵심 요약

- PostgreSQL replication lag는 primary의 WAL 진행 위치와 replica의 수신·기록·flush·replay 위치 차이로 나눠 본다.
- 시간 지표 하나만 믿지 않고 lag byte, WAL 생성률과 replica CPU·I/O를 같은 시간대에 비교한다.
- 업무별 허용 staleness를 넘은 replica는 read routing에서 제외하거나 primary로 fallback한다.

## 개념 설명

Streaming replication에서 primary가 만든 WAL은 replica로 전송되고 기록·flush된 뒤 실제 data page에 replay된다. 어느 단계가 뒤처지는지에 따라 network, disk와 replay CPU 병목의 대응이 달라진다.

Primary의 `pg_stat_replication`에서 `sent_lsn`, `write_lsn`, `flush_lsn`, `replay_lsn`을 비교하면 구간을 나눌 수 있다. Byte 차이와 함께 WAL 생성 속도를 봐야 따라잡을 수 있는지 판단할 수 있다. 쓰기가 없는 시간에는 마지막 replay timestamp 기반 시간 lag가 작거나 애매하게 보일 수 있다.

## 예시

```sql
SELECT application_name, sent_lsn, replay_lsn,
       pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

`sent_lsn - replay_lsn`은 primary가 보낸 범위에서 아직 적용되지 않은 양을 보여 준다. Network 회복 뒤 backlog replay가 시작되면 replica I/O와 query가 경쟁할 수 있어 회복 시간도 관찰한다.

## 면접 답변 예시

> PostgreSQL replication lag는 WAL의 전송, write, flush와 replay 위치를 비교해 어느 단계가 뒤처졌는지 나누겠습니다. 시간 lag만 보면 쓰기가 없는 구간을 오해할 수 있어 lag byte와 WAL 생성률도 함께 봅니다. Network가 회복된 뒤에는 밀린 replay가 replica I/O와 read query를 압박할 수 있습니다. 허용 staleness를 넘은 replica는 routing에서 제외하고 read-your-writes가 필요한 요청은 primary로 fallback합니다.

## 장점

- Read replica의 stale data 범위와 예상 회복 시간을 수치화할 수 있다.
- 전송·flush·replay 중 어느 단계가 병목인지 구분해 대응할 수 있다.

## 단점

- Replica의 장기 query와 resource 부족이 WAL replay와 충돌할 수 있다.
- Lagging replica를 계속 routing하면 사용자에게 오래된 상태와 일관성 오류가 보인다.

## 주의사항 / 실무 팁

- 업무별 허용 staleness와 read-your-writes 요구를 routing 정책에 반영한다.
- Lag byte, WAL 생성률, replica CPU·I/O와 예상 catch-up 시간을 같은 dashboard에서 본다.
