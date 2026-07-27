# nullable 컬럼 제거를 안전하게 배포하는 법 정리

## 핵심 요약

- Nullable 컬럼을 `NOT NULL`로 바꿀 때는 writer 배포, backfill, 검증과 최종 schema 변경을 나눠 진행한다.
- PostgreSQL에서는 `NOT VALID` check constraint와 `VALIDATE`로 기존 데이터 검사를 최종 metadata 변경과 분리할 수 있다.
- Backfill 중 새 NULL이 계속 들어오지 않는지와 replica lag·WAL 증가를 함께 관찰한다.

## 개념 설명

Nullable 허용을 제거하려면 기존 NULL을 채우는 것과 앞으로 NULL이 들어오지 않게 만드는 일을 모두 해결해야 한다. 구버전 application이 계속 NULL을 쓰는 상태에서 제약을 먼저 적용하면 rolling 배포 중 요청이 실패한다.

먼저 새 writer가 값을 쓰게 배포하고 `CHECK (column IS NOT NULL) NOT VALID`를 추가한다. 작은 batch로 backfill한 뒤 constraint를 validate하면 긴 기존 행 검사를 별도 단계에서 수행할 수 있다. DB version이 검증된 check constraint를 이용해 `SET NOT NULL`의 추가 scan을 줄이는지도 실제 실행 계획과 lock으로 확인한다.

## 예시

```sql
ALTER TABLE users ADD CONSTRAINT users_email_nn
CHECK (normalized_email IS NOT NULL) NOT VALID;
ALTER TABLE users VALIDATE CONSTRAINT users_email_nn;
ALTER TABLE users ALTER COLUMN normalized_email SET NOT NULL;
```

Constraint는 추가된 뒤 새·변경 행에는 적용되므로 backfill 중 새 NULL 유입도 막을 수 있다. 그래도 writer의 오류율과 NULL count를 확인하고 최종 전환 전 rollback 순서를 준비한다.

## 면접 답변 예시

> Nullable 허용 제거는 writer 배포, constraint 추가, backfill, 검증과 `SET NOT NULL`을 단계로 나누겠습니다. 먼저 모든 application version이 값을 쓰게 한 뒤 `NOT VALID` check constraint로 새 NULL을 막고, 기존 데이터는 작은 batch로 채웁니다. `VALIDATE CONSTRAINT`와 NULL count가 통과한 뒤 최종 metadata 변경을 수행합니다. 각 단계에서 replica lag와 lock wait를 보고 중단·rollback 조건을 적용합니다.

## 장점

- 혼재 version이 있는 동안 새 NULL 유입과 writer 오류를 관찰할 수 있다.
- 기존 행 검증과 최종 schema 변경을 분리해 lock 위험을 줄일 수 있다.

## 단점

- Backfill은 WAL과 replica lag, table bloat를 크게 늘릴 수 있다.
- 구버전 writer가 남아 있으면 constraint 오류가 발생하거나 전환 일정이 지연된다.

## 주의사항 / 실무 팁

- 작은 batch로 backfill하고 replica lag, lock wait와 DB 부하에 따라 속도를 낮추거나 멈춘다.
- Constraint 이름, 제거 순서와 application rollback 가능 시점을 migration runbook에 적는다.
