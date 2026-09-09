# Lamport Clock 정리

## 핵심 요약

- 분산 event에 인과성을 보존하는 순번을 붙인다.
- concurrent 여부를 직접 알려 주지 않는다.
- counter와 node id를 함께 ordering key로 쓴다.

## 개념 설명

Lamport clock은 각 process의 증가 counter로 event의 happened-before 관계를 보존하는 scalar logical clock이다.

local event마다 counter를 증가시키고 message 수신 시 max(local, received)+1로 갱신하면 원인 event의 값이 결과 event보다 작아진다.

## 예시

```text
A counter=5 -> send(ts=6)
B counter=3 receives 6 -> counter=max(3,6)+1=7
```

두 timestamp의 크기만으로 인과 관계를 역추론할 수 없고 concurrent event도 임의 순서로 정렬될 수 있다.

## 면접 답변 예시

> Lamport clock은 각 process의 counter를 이용해 원인 event의 값이 결과 event보다 작도록 만드는 논리 시계입니다. Message를 받을 때 `max(local, received) + 1`로 갱신하면 물리 clock skew 없이 happened-before 관계를 보존할 수 있습니다. 하지만 timestamp가 작다는 사실만으로 실제 인과 관계를 역으로 확정할 수 없고 concurrent event도 구분하지 못합니다. 전순서가 필요하면 node ID를 tie-break로 더하고 충돌 탐지가 목적이면 vector clock 같은 구조를 검토하겠습니다.

## 장점

- 물리 clock skew의 영향을 받지 않는다.

## 단점

- counter persistence 실패 시 값이 뒤로 갈 수 있다.

## 주의사항 / 실무 팁

- 재시작 뒤 counter 단조성을 보존한다.
