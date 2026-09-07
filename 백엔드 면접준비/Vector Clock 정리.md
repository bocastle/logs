# Vector Clock 정리

## 핵심 요약

- 동시 update와 인과 순서를 구분한다.
- node 수가 많으면 metadata가 커진다.
- aggregate나 replica 집합 단위로 vector를 제한한다.

## 개념 설명

vector clock은 참여 node별 counter를 모아 두 event의 happened-before 관계와 concurrent 변경을 구분하는 논리 시계다.

모든 component가 작거나 같고 하나 이상 작으면 선행 관계이며 서로 우세하지 않으면 concurrent로 판단해 merge 또는 충돌 처리를 한다.

## 예시

```text
A={A:3,B:1}
B={A:2,B:4}
A와 B는 서로 dominate하지 않음 -> concurrent -> merge
```

vector 크기는 참여자 수에 비례하므로 node가 많거나 자주 바뀌는 시스템에서는 version vector 압축 정책이 필요하다.

## 면접 답변 예시

> Vector clock은 참여 node별 counter를 기록해 두 version 사이의 인과 관계와 동시 변경을 구분하는 논리 시계입니다. 한 vector의 모든 값이 작거나 같고 하나 이상 작으면 선행 관계이고, 서로 우세하지 않으면 concurrent로 판단합니다. 물리 clock보다 충돌 탐지에는 안전하지만 어떤 값을 이길지 결정하는 business merge 규칙까지 제공하지는 않습니다. 참여자가 늘수록 metadata가 커지므로 replica 범위를 제한하고 삭제 tombstone과 오래된 counter 정리 절차도 같이 설계합니다.

## 장점

- offline write 충돌을 탐지한다.

## 단점

- node identity 재사용은 counter 의미를 깨뜨린다.

## 주의사항 / 실무 팁

- concurrent case의 merge 정책을 명시한다.
