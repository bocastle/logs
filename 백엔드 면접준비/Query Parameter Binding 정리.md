# Query Parameter Binding 정리

## 핵심 요약

- 사용자 값의 SQL injection을 차단한다.
- 잘못된 type binding은 implicit cast로 index를 못 쓰게 한다.
- 값은 항상 binding하고 identifier는 allowlist로 고른다.

## 개념 설명

query parameter binding은 SQL 값 자리에 placeholder를 두고 driver가 type과 값을 별도로 전송하는 실행 방식이다.

DB는 SQL 구조와 data를 분리해 parse하므로 사용자 입력이 구문으로 실행되지 않아 SQL injection을 막고 statement 재사용을 돕는다.

## 예시

```java
PreparedStatement ps = connection.prepareStatement(
    "SELECT * FROM users WHERE email = ?");
ps.setString(1, email);
```

table명이나 ORDER BY 방향 같은 identifier는 bind parameter가 될 수 없으므로 allowlist로 선택해야 한다.

## 면접 답변 예시

> Parameter binding은 SQL 구조와 사용자 값을 분리해 입력이 query 문법으로 실행되지 않게 하는 기본적인 SQL injection 방어입니다. 값은 placeholder로 binding하되 table명이나 정렬 방향 같은 identifier는 bind할 수 없으므로 정해진 allowlist에서 선택해야 합니다. Column type과 다른 type으로 binding하면 implicit cast 때문에 index를 못 쓸 수 있고, 큰 `IN` 목록은 bind 수와 실행 plan 품질을 따로 살펴야 합니다. Prepared statement 재사용은 driver와 DB 설정에 따라 달라질 수 있으니 성능 이점은 측정하고 log의 민감한 bind 값은 마스킹하겠습니다.

## 장점

- type 정보를 driver에 명확히 전달한다.

## 단점

- 동적 identifier 문자열 결합은 여전히 주입에 취약하다.

## 주의사항 / 실무 팁

- DB column type과 setter type을 맞춘다.
