# Database Row Level Security 정리

## 핵심 요약

- tenant filter 누락을 DB에서 방어한다.
- 복잡한 policy는 query plan과 latency에 영향을 준다.
- application role에 BYPASSRLS와 owner 권한을 주지 않는다.

## 개념 설명

Row Level Security(RLS)는 database가 현재 role과 session context에 따라 row별 접근 policy를 자동 적용하는 보안 기능이다.

PostgreSQL은 USING policy로 조회·삭제 대상, WITH CHECK로 삽입·변경 결과를 제한하고 current_setting의 tenant context를 predicate에 사용할 수 있다.

## 예시

```sql
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_rows ON invoices
USING (tenant_id=current_setting('app.tenant_id')::uuid)
WITH CHECK (tenant_id=current_setting('app.tenant_id')::uuid);
```

table owner와 BYPASSRLS role은 policy를 우회할 수 있어 application connection은 별도 최소 권한 role이어야 한다.

## 면접 답변 예시

> PostgreSQL RLS는 application query에 tenant predicate가 빠져도 database가 role과 session context로 row 접근을 제한하는 방어선입니다. `USING`은 읽기·삭제 대상, `WITH CHECK`는 insert와 update 결과를 제한하므로 둘을 각각 test하겠습니다. Table owner와 `BYPASSRLS` role은 policy를 우회할 수 있어 runtime connection에는 별도 최소 권한 role을 사용합니다. Pool connection마다 transaction 시작에 tenant context를 `SET LOCAL`하고 누락·잘못된 tenant·cross-tenant write 시나리오를 침투 test합니다.

## 장점

- 여러 query path에 같은 row policy를 적용한다.

## 단점

- session context 초기화 누락은 잘못된 tenant를 사용한다.

## 주의사항 / 실무 팁

- transaction 시작마다 tenant context를 SET LOCAL한다.
