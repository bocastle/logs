# Database Isolation Level 선택 기준 정리

## 핵심 요약

- 업무가 허용하는 동시성 anomaly를 명시한다.
- 강한 수준은 abort와 lock·version 보관 비용을 늘린다.
- non-repeatable read와 phantom read 시나리오를 재현한다.

## 개념 설명

isolation level은 concurrent transaction이 서로의 중간·변경 결과를 어느 범위까지 볼 수 있는지 정하는 database 일관성 수준이다.

PostgreSQL에서는 READ COMMITTED가 statement별 snapshot을, REPEATABLE READ가 transaction snapshot을 사용하고 SERIALIZABLE이 위험한 dependency를 감지해 transaction을 abort할 수 있다. 같은 isolation 이름의 세부 보장은 DB 제품마다 다르다.

## 예시

```text
READ COMMITTED: 같은 row 재조회 값이 바뀔 수 있음
REPEATABLE READ: snapshot 고정
SERIALIZABLE: 위험한 의존성 감지 후 한 transaction abort
```

가장 높은 수준을 일괄 적용하기보다 재고 차감, 보고서 조회처럼 anomaly 허용 여부와 retry 가능성을 use case별로 결정한다.

## 면접 답변 예시

> Isolation level은 가장 강한 값을 일괄 적용하기보다 업무가 허용하지 않는 anomaly를 기준으로 선택하겠습니다. PostgreSQL의 READ COMMITTED와 REPEATABLE READ snapshot 범위, SERIALIZABLE의 abort 가능성을 실제 동시 transaction 시나리오로 검증합니다. 강한 수준은 conflict와 retry 비용을 늘리고 같은 이름도 DB마다 동작이 다릅니다. Serialization failure를 정상적인 재시도 결과로 다루되 transaction 밖 외부 부작용은 멱등하게 만들어 중복을 막습니다.

## 장점

- 필요한 구간만 강한 일관성을 적용한다.

## 단점

- DB 제품마다 같은 이름의 세부 동작이 다르다.

## 주의사항 / 실무 팁

- 실제 DB 문서의 isolation 구현을 확인한다.
