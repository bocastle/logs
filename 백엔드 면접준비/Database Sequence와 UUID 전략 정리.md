# Database Sequence와 UUID 전략 정리

## 핵심 요약

- sequence는 작고 정렬 친화적인 primary key를 만든다.
- sequence 값은 외부에서 규모를 추측하게 한다.
- UUID는 native binary type으로 저장한다.

## 개념 설명

database sequence는 중앙 증가 숫자를 발급하고 UUID는 노드가 충돌 가능성이 매우 낮은 식별자를 독립 생성하는 key 전략이다.

sequence와 UUIDv7은 증가 방향이 있어 B-tree locality가 좋지만 UUIDv4는 random insert로 page split과 index 크기를 늘릴 수 있다.

## 예시

```text
sequence BIGINT: 10041, 10042, 10043
UUIDv4: random 128-bit
UUIDv7: timestamp prefix + random bits
```

외부 노출 추측 방지와 분산 생성이 필요하면 UUID가 유리하고, 작은 index와 단순 정렬이 중요하면 sequence가 유리하다.

## 면접 답변 예시

> Sequence는 작은 정수 key와 좋은 B-tree locality가 장점이라 단일 DB 중심 시스템에 단순하고 효율적입니다. 반면 UUID는 여러 node가 DB 왕복 없이 식별자를 만들고 외부에 연속 번호를 노출하지 않을 수 있습니다. UUIDv4의 random insert 비용이 문제라면 시간 순서 특성이 있는 UUIDv7을 검토하되, 시간순 정렬과 완전한 생성 순서 보장은 구분하겠습니다. DB의 native UUID나 적절한 binary type에 저장하고 실제 insert p99, index 크기와 외부 노출 요구를 기준으로 선택합니다.

## 장점

- UUID는 DB 왕복 없이 여러 node에서 생성한다.

## 단점

- UUIDv4는 index page split을 늘린다.

## 주의사항 / 실무 팁

- 외부 public id와 내부 key를 분리할지 검토한다.
