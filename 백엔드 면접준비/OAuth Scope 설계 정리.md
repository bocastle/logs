# OAuth Scope 설계 정리

## 핵심 요약

- 최소 권한 token을 발급하기 쉽다.
- scope를 너무 크게 만들면 token 하나가 과도한 권한을 가진다.
- 읽기와 쓰기 scope를 분리한다.

## 개념 설명

OAuth Scope는 access token이 어떤 API 행동을 할 수 있는지 표현하는 권한 문자열이다.

scope는 자원과 행동을 기준으로 `orders:read`, `orders:write`처럼 작게 나누고, tenant나 row 권한은 별도 authorization 정책에서 확인한다.

## 예시

```text
required_scope = orders:write
token_scope = orders:read payments:read
-> reject 403 insufficient_scope
```

scope는 기능 접근권이고, 특정 주문의 소유권 같은 resource permission까지 대신하지 않는다.

## 면접 답변 예시

> OAuth scope는 access token이 어떤 API 기능을 호출할 수 있는지 나타내는 권한 범위입니다. `orders:read`와 `orders:write`처럼 resource와 action을 기준으로 최소화하고 client 종류와 사용자 동의 흐름에 필요한 scope만 발급하겠습니다. Scope가 있어도 특정 order의 소유권이나 tenant 경계까지 증명하는 것은 아니므로 API에서 resource authorization을 다시 검사합니다. 새 scope 도입과 기존 scope 제거는 client 사용량을 확인해 점진적으로 진행하고 `insufficient_scope` 응답, 필요한 scope와 감사 기준을 문서화합니다.

## 장점

- client별 접근 범위를 문서화할 수 있다.

## 단점

- tenant 권한을 scope 문자열에 모두 넣으면 관리가 복잡해진다.

## 주의사항 / 실무 팁

- resource ownership 검사는 scope 뒤에 별도로 수행한다.
