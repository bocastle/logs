# Logical Replication 정리

## 핵심 요약

- 필요한 table만 선택해 복제한다.
- primary key가 없는 table update는 비싸거나 제한된다.
- publication 대상과 replica identity를 점검한다.

## 개념 설명

logical replication은 WAL에서 row 수준 변경을 해석해 publication의 table 변경을 subscription 대상에 전달하는 복제 방식이다.

publisher가 logical decoding으로 INSERT·UPDATE·DELETE를 내보내고 subscriber가 apply worker로 재실행하므로 선택적 table 복제와 버전 간 이동이 가능하다.

## 예시

```sql
CREATE PUBLICATION app_pub FOR TABLE orders;
CREATE SUBSCRIPTION app_sub
CONNECTION 'host=primary dbname=app'
PUBLICATION app_pub;
```

DDL과 sequence 상태는 일반적으로 row 변경처럼 자동 복제되지 않으므로 schema migration과 cutover에서 별도로 맞춰야 한다.

## 면접 답변 예시

> PostgreSQL logical replication은 WAL의 row 변경을 해석해 publication에 포함된 table만 subscriber로 전달하는 방식입니다. 선택 복제와 major version migration에 유용하지만 DDL과 sequence 상태는 같은 방식으로 따라오지 않으므로 별도 migration 절차가 필요합니다. UPDATE와 DELETE를 정확히 식별할 수 있도록 primary key나 적절한 replica identity도 확인하겠습니다. 운영할 때는 apply lag뿐 아니라 중단된 subscription의 replication slot이 WAL을 계속 쌓는지도 함께 감시합니다.

## 장점

- 물리 형식이 다른 버전 사이 migration에 활용한다.

## 단점

- subscription 중단 시 replication slot이 WAL을 쌓는다.

## 주의사항 / 실무 팁

- subscription lag와 slot WAL을 알림화한다.
