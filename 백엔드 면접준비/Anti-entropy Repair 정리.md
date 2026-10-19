# Anti-entropy Repair 정리

## 핵심 요약

- 장애 뒤 장기간 남은 replica divergence를 줄인다.
- repair scan과 streaming이 foreground I/O를 잠식한다.
- repair bandwidth와 concurrency를 제한한다.

## 개념 설명

anti-entropy repair는 replica들이 background에서 데이터 digest를 비교해 누락되거나 다른 key를 찾아 동기화하는 수리 과정이다.

Merkle tree로 key range hash를 단계적으로 비교해 다른 구간만 전송하고 version 또는 timestamp 규칙으로 replica 값을 reconcile한다.

## 예시

```text
range 0-999 root hash differs
compare Merkle tree children -> range 500-624 differs
stream missing versions, then read repair verifies key
```

read repair가 읽힌 key만 고치는 반면 anti-entropy는 읽히지 않는 cold key까지 주기적으로 검사한다.

## 면접 답변 예시

> Anti-entropy repair는 replica끼리 key range digest를 비교해 오래 남은 divergence를 background에서 찾고 고치는 과정입니다. Read repair가 조회된 key만 다루는 것과 달리 cold data까지 검사할 수 있고 Merkle tree로 다른 range만 좁혀 전송량을 줄입니다. Tombstone이 사라진 뒤 오래된 replica가 돌아오면 삭제 data가 부활할 수 있어 모든 replica의 최대 장애·repair 주기보다 tombstone retention을 충분히 길게 둬야 합니다. Foreground I/O를 침범하지 않도록 bandwidth와 concurrency를 제한하고 mismatch range와 repair age를 관찰합니다.

## 장점

- Merkle tree로 전체 row 전송을 피한다.

## 단점

- 잘못된 conflict rule은 최신 값을 덮을 수 있다.

## 주의사항 / 실무 팁

- tombstone retention 안에 모든 replica를 repair한다.
