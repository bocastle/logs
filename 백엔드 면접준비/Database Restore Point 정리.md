# Database Restore Point 정리

## 핵심 요약

- 중요 변경 직전 복구 위치를 명확히 이름 붙인다.
- WAL archive 누락 시 marker가 있어도 복구하지 못한다.
- 이름에 배포 id와 시각을 포함한다.

## 개념 설명

database restore point는 WAL에 사람이 식별할 이름과 LSN을 기록해 PITR의 정확한 recovery target으로 사용하는 marker다.

PostgreSQL의 pg_create_restore_point 호출을 배포나 batch 직전에 실행하고 base backup과 WAL archive에서 해당 restore point까지 replay한다.

## 예시

```sql
SELECT pg_create_restore_point('before_billing_v42');
-- recovery_target_name = 'before_billing_v42'
```

restore point는 snapshot이나 backup이 아니므로 그 LSN까지의 base backup과 연속 WAL이 없으면 복구할 수 없다.

## 면접 답변 예시

> Database restore point는 PostgreSQL WAL에 이름을 붙인 복구 marker이며 snapshot이나 backup 자체는 아닙니다. 중요 배포 직전에 고유한 이름으로 만들고 그 LSN까지 이어지는 base backup과 WAL archive가 실제로 존재하는지 확인하겠습니다. 별도 환경에서 `recovery_target_name`으로 복구해 schema와 핵심 row를 검증하고 목표 지점 이후 transaction이 제외된다는 점을 복구 의사결정에 반영합니다. 배포 ID·시각·LSN을 runbook에 연결하고 같은 이름을 재사용하지 않습니다.

## 장점

- wall clock보다 정확한 WAL recovery target을 제공한다.

## 단점

- 동일 이름 재사용은 운영 혼동을 만든다.

## 주의사항 / 실무 팁

- restore point LSN과 archive 존재를 확인한다.
