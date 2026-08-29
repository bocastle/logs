# Database Write Concern 정리

## 핵심 요약

- failover 뒤 확인된 write 손실 위험을 줄인다.
- 강한 concern은 원격 replica 지연을 쓰기 경로에 넣는다.
- timeout 재시도에는 idempotency key를 사용한다.

## 개념 설명

write concern은 분산 database가 몇 replica의 acknowledgement를 받은 뒤 write 성공을 반환할지 정하는 내구성 정책이다.

majority write concern은 voting member 과반의 기록 확인을 기다려 failover 뒤 rollback 가능성을 낮추지만 acknowledgement 대기만큼 latency가 늘어난다.

## 예시

```text
replica set=5
write concern=majority -> 3개 member acknowledgement 뒤 성공
wtimeout=2000ms
```

write concern은 journaling 여부와 함께 봐야 하며 timeout은 실패 결과가 아니라 commit 여부가 불확실한 상태일 수 있다.

## 면접 답변 예시

> Write concern은 분산 database가 몇 replica의 확인을 받은 뒤 client에 성공을 반환할지 정하는 내구성 정책입니다. Majority를 사용하면 failover 뒤 acknowledged write가 rollback될 위험을 낮출 수 있지만 replica 지연이 write latency에 포함됩니다. `wtimeout`이 발생했다고 해서 write가 확실히 실패한 것은 아니고 결과를 모르는 상태일 수 있으므로, 재시도에는 idempotency key나 결과 조회 절차가 필요합니다. 업무의 손실 허용치와 latency 목표를 기준으로 concern과 journaling을 정하고 failover 훈련으로 실제 동작을 확인하겠습니다.

## 장점

- 업무별 내구성 수준을 명시한다.

## 단점

- timeout 재시도는 중복 write를 만들 수 있다.

## 주의사항 / 실무 팁

- latency와 데이터 손실 허용치로 concern을 고른다.
