# Database Failback 전략 정리

## 핵심 요약

- 임시 DR 환경에서 정상 topology로 복귀한다.
- fencing 실패는 split-brain을 만든다.
- old primary를 wipe 또는 rewind 후 replica로 만든다.

## 개념 설명

database failback은 장애 때 승격한 새 primary에서 복구된 원래 site로 쓰기 권한과 traffic을 안전하게 되돌리는 절차다.

먼저 원래 primary를 새 primary의 replica로 재구성해 data resynchronization을 완료하고 lag 0을 확인한 뒤 write fencing, role 전환, client routing 순서로 cutback한다.

## 예시

```text
old primary A fenced and rebuilt as replica of B
A catches up -> checksum/LSN verify
stop writes briefly -> promote A -> demote B -> route clients
```

양쪽 node에 write가 열린 상태로 데이터를 합치려 하면 split-brain divergence가 커지므로 한 시점에 하나의 primary만 허용한다.

## 면접 답변 예시

> Database failback은 장애 때 승격한 primary의 변경을 보존하면서 정상 topology로 쓰기 권한을 되돌리는 절차입니다. 예전 primary를 그대로 다시 열지 않고 fence한 상태에서 wipe나 rewind 후 현재 primary의 replica로 재구성해 LSN, checksum과 lag를 검증하겠습니다. 전환 때는 write를 잠시 멈추고 한쪽만 promote한 뒤 old primary를 demote하며 DNS뿐 아니라 connection pool과 proxy route가 모두 새 역할을 보는지 확인합니다. Failback 자체가 추가 장애가 될 수 있으므로 즉시 복귀를 서두르지 않고 용량, replication 안정성과 되돌림 기준을 충족한 뒤 runbook을 실행합니다.

## 장점

- 역방향 복제를 검증한 뒤 role을 전환한다.

## 단점

- 재동기화가 network와 disk를 포화시킨다.

## 주의사항 / 실무 팁

- promotion 전 LSN과 checksum을 검증한다.
