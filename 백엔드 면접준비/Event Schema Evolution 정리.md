# Event Schema Evolution 정리

## 핵심 요약

- 과거 event replay 가능성을 유지한다.
- required field 추가는 오래된 event를 읽지 못하게 한다.
- 과거 payload corpus로 reader를 테스트한다.

## 개념 설명

event schema evolution은 이미 저장·전달된 event를 오래된 consumer와 새 consumer가 계속 해석하도록 schema를 변경하는 규율이다.

schema registry의 compatibility mode 아래 optional field 추가, default 제공, enum 확장 규칙을 적용하고 schema version별 reader fixture를 검증한다.

## 예시

```text
v1 OrderPaid {order_id, amount}
v2 OrderPaid {order_id, amount, optional currency='KRW'}
registry mode=BACKWARD_TRANSITIVE
```

문법 compatibility가 통과해도 기존 field의 단위나 의미를 바꾸면 semantic compatibility가 깨지므로 새 field를 추가해야 한다.

## 면접 답변 예시

> Event schema evolution은 이미 저장된 과거 event와 서로 다른 시점에 배포된 consumer가 계속 호환되도록 schema를 바꾸는 규율입니다. Registry의 compatibility 검사는 required field 추가 같은 구조적 파손을 막는 1차 방어이고, optional field와 default도 실제 reader 언어에서 같은 의미인지 확인해야 합니다. 기존 field의 단위나 의미를 바꾸면 문법 검사를 통과해도 조용한 데이터 오류가 생기므로 새 field나 event type으로 표현하겠습니다. 과거 payload corpus와 새 enum 값을 구 consumer에 replay하는 test를 유지합니다.

## 장점

- 독립 배포 consumer의 decode 실패를 줄인다.

## 단점

- enum 새 값은 구 consumer 분기를 깨뜨릴 수 있다.

## 주의사항 / 실무 팁

- schema version과 producer 정보를 event에 남긴다.
