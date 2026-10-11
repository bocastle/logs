# Database Role과 권한 분리 정리

## 핵심 요약

- 유출된 credential의 피해 범위를 제한한다.
- PUBLIC grant가 최소 권한을 우회할 수 있다.
- owner login을 runtime에서 금지한다.

## 개념 설명

database role 분리는 login identity와 권한 묶음을 나누고 GRANT로 업무별 최소 권한을 부여하는 least privilege 설계다.

owner, migration, application read-write, read-only, operator role을 분리하고 application login은 필요한 role만 SET ROLE 가능하게 제한한다.

## 예시

```sql
CREATE ROLE app_rw NOLOGIN;
GRANT SELECT, INSERT, UPDATE ON orders TO app_rw;
GRANT app_rw TO app_service;
```

table owner 권한으로 application을 실행하면 GRANT 검사를 우회하고 DROP·ALTER까지 가능해 침해 범위가 커진다.

## 면접 답변 예시

> Database role은 login identity와 권한 묶음을 분리해 runtime credential이 유출돼도 피해 범위를 제한하는 설계입니다. Owner와 migration, application read-write·read-only 및 operator role을 나누고 application은 owner로 접속하지 않게 하겠습니다. 개별 grant만 보지 않고 `PUBLIC`, role inheritance와 `SET ROLE` 가능한 membership까지 확인해야 간접 권한을 놓치지 않습니다. 새 object에도 정책이 이어지도록 schema와 생성 owner별 default privilege를 설정하고 실제 grant를 정기적으로 diff합니다.

## 장점

- 권한 변경을 role 단위로 감사한다.

## 단점

- role inheritance가 간접 권한을 숨긴다.

## 주의사항 / 실무 팁

- role membership과 object grant를 정기 diff한다.
