# Database Advisory Lock 정리

## 핵심 요약

- row가 없는 논리 자원도 잠글 수 있다.
- key collision은 무관한 작업을 직렬화한다.
- 가능하면 transaction-scoped advisory lock을 쓴다.

## 개념 설명

advisory lock은 database가 row와 무관한 application-defined integer key에 잠금을 제공하는 협력적 동시성 제어다.

PostgreSQL의 pg_try_advisory_lock은 즉시 성공 여부를 반환하고 pg_try_advisory_xact_lock은 transaction 종료 때 자동 해제된다.

## 예시

```sql
SELECT pg_try_advisory_xact_lock(hashtextextended('invoice:42', 0));
```

DB가 업무 자원 의미를 알지 못하므로 모든 code path가 같은 key 규칙과 lock protocol을 지켜야 효과가 있다.

## 면접 답변 예시

> PostgreSQL advisory lock은 row가 없는 논리 자원도 application이 정한 integer key로 잠그는 협력적 동시성 제어입니다. 가능하면 transaction-scoped try lock을 사용해 commit이나 rollback 때 자동 해제하고 대기 대신 선점 실패를 명확히 처리하겠습니다. DB는 key의 업무 의미를 모르므로 모든 writer가 같은 key 생성 규칙을 따라야 하며 hash collision은 무관한 작업까지 직렬화합니다. Session lock은 pool connection에 남을 위험이 있어 적용 범위를 제한하고 획득 실패율과 보유 시간을 관찰합니다.

## 장점

- try API로 대기 없이 leader 작업을 선점한다.

## 단점

- session lock 반납 누락은 pool connection에 남는다.

## 주의사항 / 실무 팁

- key 생성 함수를 공용 코드로 고정한다.
