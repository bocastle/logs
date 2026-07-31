# 운영용 backfill 작업을 안전하게 쪼개는 법 정리

## 핵심 요약

- Backfill은 작은 batch와 짧은 트랜잭션으로 나눠 lock, WAL과 replica lag를 제한한다.
- 변환 함수와 대상 조건을 표본에서 검증하고 kill switch, rate limit과 재시작 가능한 progress를 준비한다.
- 처리량보다 online traffic의 지연 budget을 우선하고 lag·lock wait 임계값에서 자동으로 멈춘다.

## 개념 설명

운영 중 대량 `UPDATE`를 한 트랜잭션으로 실행하면 긴 row lock, WAL spike와 replica 지연을 만들 수 있다. Backfill은 대상 범위를 작은 batch로 선택해 commit 사이에 부하를 다시 확인하는 작업이다.

PostgreSQL에서는 CTE와 `FOR UPDATE SKIP LOCKED`로 여러 worker가 다른 row를 가져가게 할 수 있다. 하지만 worker 수를 늘리는 것이 항상 빠른 것은 아니며 index scan과 WAL·vacuum 부담도 함께 커진다. 변환 로직은 재실행해도 결과가 같고 이미 처리한 row를 조건에서 제외해야 한다.

## 예시

```sql
WITH batch AS (
  SELECT id FROM users
  WHERE normalized_email IS NULL
  ORDER BY id
  LIMIT 5000
  FOR UPDATE SKIP LOCKED
)
UPDATE users u
SET normalized_email = lower(u.email)
FROM batch
WHERE u.id = batch.id;
```

Batch마다 commit하고 처리 row 수, 오류와 변환 결과를 기록한다. Replica lag나 lock wait, DB CPU가 임계값을 넘으면 sleep하거나 kill switch로 멈춘다. `SKIP LOCKED`로 계속 남는 row는 마지막 별도 pass에서 확인한다.

## 면접 답변 예시

> 운영 backfill은 작은 batch와 짧은 트랜잭션으로 나눠 online traffic 영향을 제한하겠습니다. 대상 조건과 변환 함수를 표본에서 먼저 검증하고, 재실행해도 같은 결과가 되게 만듭니다. 여러 worker는 `SKIP LOCKED`로 분담할 수 있지만 replica lag, lock wait와 DB 부하가 기준을 넘으면 자동으로 속도를 낮추거나 멈춥니다. 처리 progress와 kill switch를 운영 상태로 남기고 마지막에는 누락 row와 checksum을 확인합니다.

## 장점

- 처리 조건이 멱등하면 중단 뒤 남은 row부터 안전하게 재시작할 수 있다.
- Batch별 영향과 변환 오류를 작게 제한해 조기에 중단할 수 있다.

## 단점

- `SKIP LOCKED`만 믿으면 계속 충돌하는 row가 마지막까지 남을 수 있다.
- 잘못된 변환을 여러 worker로 빠르게 대량 확산할 위험이 있다.

## 주의사항 / 실무 팁

- 갱신 전후 NULL count, 대상 row 수와 checksum을 기록한다.
- 마지막 pass에서는 skip된 row와 online writer가 다시 이전 값을 만든 사례를 별도로 확인한다.
