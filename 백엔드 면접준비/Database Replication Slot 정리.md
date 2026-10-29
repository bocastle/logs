# Database Replication Slot 정리

## 핵심 요약

- consumer disconnect 중에도 필요한 WAL을 보존한다.
- inactive slot이 WAL disk를 무제한 사용할 수 있다.
- restart_lsn retained bytes에 강한 알림을 둔다.

## 개념 설명

replication slot은 PostgreSQL이 consumer가 아직 읽지 않은 WAL과 필요한 row version을 보존하도록 restart_lsn을 추적하는 서버 객체다.

logical slot의 confirmed_flush_lsn이 진전하면 restart_lsn도 이동할 수 있고, consumer가 멈추면 WAL retention이 계속 늘어난다.

## 예시

```sql
SELECT slot_name, active, restart_lsn, confirmed_flush_lsn,
       pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) AS retained_bytes
FROM pg_replication_slots;
```

slot은 consumer 데이터가 아니라 WAL 보관 약속이므로 장기 비활성 slot을 방치하면 primary disk를 가득 채울 수 있다.

## 면접 답변 예시

> PostgreSQL replication slot은 consumer가 아직 처리하지 않은 WAL과 필요한 row version을 server가 보존하게 하는 약속입니다. 일시 disconnect 후 같은 위치에서 이어갈 수 있지만 inactive slot의 `restart_lsn`이 멈추면 primary disk가 계속 늘어납니다. Slot별 owner, retained bytes와 최대 비활성 시간을 강하게 alert하고 단순 삭제가 미소비 변경을 잃는다는 점을 runbook에 명시하겠습니다. Failover 환경에서 slot이 어떻게 승계되는지 확인하고 복구 불가 시 새 snapshot부터 재동기화하는 절차를 준비합니다.

## 장점

- CDC resume 위치를 서버가 추적한다.

## 단점

- slot 삭제는 미소비 변경을 잃게 한다.

## 주의사항 / 실무 팁

- slot owner와 폐기 기준을 inventory에 기록한다.
