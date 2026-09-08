# DB Connection Validation Query 정리

## 핵심 요약

- 방화벽이 끊은 stale connection 재사용을 줄인다.
- 과도한 검증은 DB QPS를 늘린다.
- 가능하면 표준 isValid를 사용한다.

## 개념 설명

connection validation query는 pool에서 빌려 줄 connection이 database와 통신 가능한지 확인하는 가벼운 SQL 검사다.

JDBC 4 driver는 Connection.isValid(timeout)을 우선 사용할 수 있고, 지원이 불완전할 때 SELECT 1 같은 validation query를 checkout 또는 idle 검사에 실행한다.

## 예시

```text
validation: Connection.isValid(2)
fallback validation query: SELECT 1
validation timeout: 2s
```

매 checkout마다 원격 SELECT 1을 실행하면 DB round trip이 요청 latency와 QPS에 추가되므로 driver 검증과 idle keepalive를 구분한다.

## 면접 답변 예시

> Connection validation은 pool이 stale connection을 application request에 넘기기 전에 통신 가능 여부를 가볍게 확인하는 절차입니다. JDBC driver가 지원하면 `Connection.isValid()`를 우선하고, 필요할 때만 `SELECT 1` 같은 query를 사용하겠습니다. 매 checkout마다 원격 query를 실행하면 그 round trip이 모든 request latency와 DB QPS에 더해지므로 idle validation, keepalive와 max lifetime을 network 정책에 맞춰 선택해야 합니다. 검증 직후 실제 query도 실패할 수 있어 정상적인 timeout과 retry 설계는 별도로 유지합니다.

## 장점

- 장애 connection을 요청 전에 pool에서 제거한다.

## 단점

- 무거운 query는 정상 connection도 늦게 반환한다.

## 주의사항 / 실무 팁

- validation timeout을 업무 query timeout보다 짧게 둔다.
