# DB Prepared Statement 캐시 무효화 정리

## 핵심 요약

- stale plan이 변경된 schema를 잘못 해석하는 일을 막는다.
- 대규모 invalidation 직후 parse CPU와 latency가 튄다.
- DDL 배포 뒤 prepare error와 parse latency를 본다.

## 개념 설명

prepared statement plan invalidation은 DDL이나 의존 객체 변경으로 cached plan을 더 이상 안전하게 재사용할 수 없어 폐기하는 동작이다.

DB는 table·type·function dependency를 추적해 DDL 뒤 plan invalidation과 reparse를 수행하고, connection pool의 session별 cache는 각 연결에서 reprepare해야 한다.

## 예시

```sql
ALTER TABLE orders ALTER COLUMN status TYPE varchar(32);
DEALLOCATE find_orders;
PREPARE find_orders(text) AS SELECT * FROM orders WHERE status=$1;
```

결과 column type이 바뀌면 구 cached plan 실행에서 오류가 날 수 있으므로 migration과 application statement cache 수명을 함께 조정한다.

## 면접 답변 예시

> Prepared statement cache invalidation은 DDL이나 의존 객체 변경 뒤 기존 statement나 plan을 더 이상 안전하게 재사용할 수 없을 때 폐기하는 동작입니다. DB가 의존성을 추적해 자동으로 다시 plan을 만들더라도 driver와 connection별 statement cache가 별도로 남을 수 있어 배포 조합을 확인해야 합니다. Schema 변경 직후 prepare error와 parse latency가 튀면 해당 cache를 비우고 한 번만 reprepare하되, 같은 오류를 무한 재시도하지 않겠습니다. 큰 migration에서는 pool connection이 한꺼번에 reprepare하며 CPU가 치솟지 않는지도 관찰합니다.

## 장점

- DDL 의존성을 기준으로 필요한 plan만 다시 만든다.

## 단점

- driver cache가 DB invalidation을 늦게 인식할 수 있다.

## 주의사항 / 실무 팁

- schema 변경 시 pool connection의 reprepare 전략을 검증한다.
