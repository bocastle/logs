# Read Model Projection 정리

## 핵심 요약

- 조회 join을 미리 계산해 latency를 낮춘다.
- projection lag 동안 stale view를 제공한다.
- checkpoint와 upsert를 같은 transaction에 둔다.

## 개념 설명

read model projection은 event stream을 소비해 화면과 query에 최적화된 파생 table을 만드는 변환 과정이다.

projector가 event를 순서대로 적용하고 offset checkpoint와 read model upsert를 원자적으로 저장하면 재시작과 중복 delivery를 견딘다.

## 예시

```text
event offset=1042 OrderPaid(order=42)
transaction: order_summary upsert + checkpoint=1042
```

projection은 원본이 아니므로 version이 바뀌면 새 table에 전체 replay한 뒤 routing을 전환할 수 있어야 한다.

## 면접 답변 예시

> Read model projection은 event stream을 소비해 화면 조회에 맞는 파생 table을 미리 만드는 과정입니다. Projector가 read model upsert와 offset checkpoint를 같은 transaction에 저장하면 crash 뒤 중복 delivery를 안전하게 다시 처리할 수 있습니다. Projection은 원본이 아니므로 logic이나 schema가 바뀌면 과거 event를 새 version의 shadow table에 replay하고 결과를 비교한 뒤 routing을 전환하겠습니다. Lag와 실패 offset을 관찰하고 handler의 비멱등 누적이 replay에서 값을 두 번 더하지 않는지 테스트합니다.

## 장점

- 화면별 schema를 write model과 독립 최적화한다.

## 단점

- 비멱등 handler는 replay 때 값을 중복 누적한다.

## 주의사항 / 실무 팁

- projection lag와 실패 offset을 알림화한다.
