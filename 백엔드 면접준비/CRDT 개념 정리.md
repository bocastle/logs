# CRDT 개념 정리

## 핵심 요약

- network partition 중에도 local update를 허용한다.
- metadata와 tombstone이 계속 커질 수 있다.
- merge 법칙을 property test로 검증한다.

## 개념 설명

CRDT는 replica가 coordination 없이 update한 상태를 deterministic하게 merge해 eventual convergence를 보장하는 데이터 타입이다.

state-based CRDT의 merge는 commutative, associative, idempotent한 join이어야 하고 operation-based 방식은 전달 조건과 causal order를 요구한다.

## 예시

```text
G-Counter A={A:3,B:1}, B={A:2,B:4}
merge component-wise max -> {A:3,B:4}, value=7
```

CRDT가 업무 충돌을 없애는 것은 아니며 삭제, 재고 하한, 유일성처럼 invariant가 필요한 규칙은 추가 coordination이 필요하다.

## 면접 답변 예시

> CRDT는 network partition 중 각 replica가 update한 값을 coordination 없이 deterministic하게 merge해 eventual convergence를 얻는 data type입니다. State-based merge는 commutative, associative와 idempotent 법칙을 만족해야 message 순서와 중복에 견딜 수 있습니다. 하지만 convergence가 재고 하한이나 username uniqueness 같은 business invariant까지 보장하지는 않아 필요한 규칙에는 coordination을 추가해야 합니다. Merge 의미를 업무와 먼저 합의하고 tombstone·causal metadata의 안전한 정리 조건을 property test로 검증합니다.

## 장점

- message 중복과 순서 변화에도 같은 상태로 수렴한다.

## 단점

- 업무 invariant를 모든 CRDT가 보장하지 않는다.

## 주의사항 / 실무 팁

- 업무가 허용하는 conflict 의미를 먼저 정한다.
