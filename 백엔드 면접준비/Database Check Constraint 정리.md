# Database Check Constraint 정리

## 핵심 요약

- 모든 writer에 동일한 도메인 규칙을 강제한다.
- 복잡한 함수 호출은 write latency를 높인다.
- immutable하고 row-local한 식을 사용한다.

## 개념 설명

check constraint는 각 row가 지정한 boolean 식을 만족하도록 database가 INSERT와 UPDATE에서 강제하는 데이터 규칙이다.

PostgreSQL은 NOT VALID로 기존 table scan 없이 새 write부터 검사하고 이후 VALIDATE CONSTRAINT로 기존 row를 online 검증할 수 있다.

## 예시

```sql
ALTER TABLE orders ADD CONSTRAINT amount_positive
CHECK (amount >= 0) NOT VALID;
ALTER TABLE orders VALIDATE CONSTRAINT amount_positive;
```

CHECK 식에서 NULL 결과는 통과하므로 필수 값이면 NOT NULL 제약을 별도로 둬야 한다.

## 면접 답변 예시

> Check constraint는 모든 writer가 같은 row-level 도메인 규칙을 지키도록 database 저장 경계에서 검사하는 장치입니다. 예를 들어 금액이 음수가 될 수 없다는 규칙은 application validation과 별개로 DB에서도 막겠습니다. PostgreSQL에서는 `NOT VALID`로 새 write부터 제약을 적용한 뒤 기존 위반 row를 정리하고 `VALIDATE CONSTRAINT`를 실행해 큰 배포를 단계화할 수 있습니다. CHECK 식의 `NULL` 결과는 통과하므로 필수 여부는 `NOT NULL`로 따로 표현하고 expression은 immutable하고 row-local하게 유지합니다.

## 장점

- NOT VALID로 큰 table 배포를 단계화한다.

## 단점

- NULL 의미를 놓치면 예상 밖 row가 통과한다.

## 주의사항 / 실무 팁

- 위반 row를 먼저 조회해 정리한다.
