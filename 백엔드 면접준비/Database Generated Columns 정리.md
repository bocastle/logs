# Database Generated Columns 정리

## 핵심 요약

- 여러 writer가 같은 파생 규칙을 공유한다.
- 원본 update마다 expression 계산 비용이 붙는다.
- immutable expression만 사용한다.

## 개념 설명

generated column은 다른 column의 deterministic expression으로 값을 계산해 database가 일관되게 유지하는 파생 column이다.

STORED 방식은 INSERT·UPDATE 때 expression을 계산해 물리 저장하므로 조회는 빠르지만 원본 변경마다 쓰기와 저장 비용이 든다.

## 예시

```sql
ALTER TABLE users ADD COLUMN normalized_email text
GENERATED ALWAYS AS (lower(email)) STORED;
CREATE INDEX ON users(normalized_email);
```

volatile function이나 다른 row를 참조하는 계산은 generated column에 부적합하며 expression 변경에는 table rewrite 가능성을 확인해야 한다.

## 면접 답변 예시

> Generated column은 다른 column에서 계산되는 파생 값을 database가 일관되게 관리하는 기능입니다. 여러 application writer가 normalized email 같은 값을 각자 계산하는 dual-write 불일치를 줄이고, stored 결과에는 index를 만들 수 있습니다. 대신 원본 update마다 계산과 저장 비용이 들고 허용되는 expression과 virtual·stored 지원은 DB마다 다릅니다. 큰 table에서 식을 추가하거나 바꿀 때는 실제 DDL lock과 table rewrite 여부, 쓰기 QPS와 저장 증가량을 staging에서 확인하겠습니다.

## 장점

- STORED 결과에 index를 만들 수 있다.

## 단점

- 복잡한 expression 변경은 큰 rewrite를 만들 수 있다.

## 주의사항 / 실무 팁

- 쓰기 QPS와 저장 증가량을 계산한다.
