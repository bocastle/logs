# Database Trigger 장단점 정리

## 핵심 요약

- 여러 application writer에 같은 규칙을 강제한다.
- 숨은 추가 SQL이 write latency를 높인다.
- trigger 함수와 실행 시간을 DB 지표로 노출한다.

## 개념 설명

database trigger는 table event에 반응해 BEFORE 또는 AFTER 시점에 database 내부 함수를 자동 실행하는 기능이다.

BEFORE trigger는 NEW row를 검증·정규화하고 AFTER trigger는 확정된 row를 audit table에 기록할 수 있으며 실행은 원래 transaction에 포함된다.

## 예시

```sql
CREATE TRIGGER orders_audit
AFTER INSERT OR UPDATE ON orders
FOR EACH ROW EXECUTE FUNCTION write_order_audit();
```

database trigger 로직은 모든 writer에 적용되는 장점이 있지만 application trace에서 숨기 쉬워 recursive trigger와 write amplification을 관리해야 한다.

## 면접 답변 예시

> Database trigger는 table event에 반응해 원래 transaction 안에서 자동 실행되는 DB 로직입니다. 여러 application writer에 동일한 검증이나 audit 기록을 강제할 때 유용하지만 호출부 코드와 trace에서 추가 write가 숨기 쉽습니다. BEFORE와 AFTER 중 필요한 시점을 고르고 `OLD`·`NEW` 값, recursion과 bulk write의 amplification을 테스트하겠습니다. 외부 API 호출 같은 복잡한 부작용은 trigger에 넣지 않고 schema와 application의 rolling 배포 순서도 호환되게 설계합니다.

## 장점

- 원래 write와 audit 기록을 한 transaction에 묶는다.

## 단점

- trigger chain과 recursion은 원인 추적을 어렵게 한다.

## 주의사항 / 실무 팁

- 복잡한 외부 부작용은 trigger 밖으로 뺀다.
