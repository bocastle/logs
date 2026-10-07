# Distributed Transaction Log 정리

## 핵심 요약

- coordinator 장애 뒤 transaction 결정을 복구한다.
- log storage 장애가 전체 transaction을 막는다.
- decision을 메시지보다 먼저 durable flush한다.

## 개념 설명

distributed transaction log는 coordinator가 여러 participant의 prepare·commit 진행 상태를 durable하게 기록해 장애 뒤 결정을 복구하는 원장이다.

global transaction id별 participant vote와 최종 결정을 먼저 log에 flush한 뒤 메시지를 보내면 coordinator 재시작 후 in-doubt transaction을 이어 처리할 수 있다.

## 예시

```text
tx=9 participants=[inventory,payment]
prepare: inventory=yes payment=yes
durable decision=commit
commit acknowledgement: inventory=yes payment=pending
```

log가 있다고 원자성이 자동 보장되는 것은 아니며 participant의 idempotent prepare·commit 처리와 decision retention이 필요하다.

## 면접 답변 예시

> Distributed transaction log는 coordinator가 participant의 prepare vote와 최종 commit·abort 결정을 durable하게 남겨 장애 뒤에도 같은 결정을 이어가는 원장입니다. 최종 결정을 participant에 보내기 전에 log를 flush해야 재시작 후 상반된 결정을 내리지 않습니다. Log만으로 원자성이 완성되는 것은 아니어서 participant는 global transaction ID로 prepare와 commit command를 멱등 처리해야 합니다. In-doubt age와 미응답 participant를 관찰하고 가장 늦게 복구되는 participant보다 decision retention을 길게 유지합니다.

## 장점

- participant별 미완료 단계를 추적한다.

## 단점

- participant lock이 prepare 상태에서 오래 유지된다.

## 주의사항 / 실무 팁

- participant command를 global tx id로 멱등 처리한다.
