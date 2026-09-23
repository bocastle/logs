# Multi-tenant 데이터 격리 방식 정리

## 핵심 요약

- 격리 모델별 routing 책임을 명확히 한다.
- session context 초기화 누락은 교차 tenant 접근을 만든다.
- transaction 시작마다 tenant context를 설정한다.

## 개념 설명

Multi-tenant 격리 방식은 shared schema, schema per tenant, database per tenant를 실제 routing과 migration 단위로 구현하는 세 가지 모델이다.

shared schema는 tenant context를 SQL predicate와 RLS에 주입하고, schema per tenant는 search_path를, database per tenant는 datasource registry를 요청마다 선택한다.

## 예시

```text
shared schema -> SET LOCAL app.tenant_id
schema per tenant -> SET LOCAL search_path=tenant_42
database per tenant -> datasource[tenant_42]
```

connection pool 재사용 시 tenant context나 search_path를 반드시 초기화하지 않으면 다음 요청으로 tenant 상태가 새어 나간다.

## 면접 답변 예시

> Multi-tenant 격리는 shared schema, tenant별 schema와 tenant별 database 중 보안·운영 비용에 맞는 경계를 고르는 문제입니다. Shared schema는 `tenant_id` predicate와 RLS를 함께 써 누락을 방어하고, schema나 database 분리는 routing과 migration 대상 수가 늘어나는 대신 격리 범위를 키웁니다. Connection pool을 재사용할 때 tenant context나 `search_path`가 다음 요청으로 새지 않도록 transaction 시작에 설정하고 반납 전에 reset하겠습니다. Tenant 이름을 SQL identifier로 직접 조합하지 않고 검증된 mapping을 사용하며 migration 결과는 전체 tenant inventory와 대조합니다.

## 장점

- tenant onboarding 자동화 범위를 정한다.

## 단점

- schema per tenant는 객체 수와 migration 시간이 늘어난다.

## 주의사항 / 실무 팁

- pool 반납 전에 session state를 reset한다.
