# Clock Skew와 분산 시스템 정리

## 핵심 요약

- skew 예산을 두면 시간 기반 정책의 안전 범위를 계산한다.
- 큰 skew는 유효 token을 조기 만료시킨다.
- offset과 drift를 node별로 모니터링한다.

## 개념 설명

clock skew는 분산 node의 wall clock이 같은 실제 순간을 서로 다른 timestamp로 나타내는 차이다.

NTP는 drift를 줄이지만 완전히 없애지 못하므로 lease 만료, token 검증, last-write-wins를 단일 wall clock 비교에만 의존하면 안 된다.

## 예시

```text
node A=12:00:00.900 node B=12:00:00.200
lease expires_at=12:00:00.500
A는 만료, B는 유효로 판단할 수 있음
```

duration은 monotonic clock, ownership은 fencing token, event ordering은 logical clock처럼 목적별 시간 기준을 분리한다.

## 면접 답변 예시

> Clock skew는 같은 실제 순간을 node마다 다른 wall-clock timestamp로 보는 차이입니다. NTP로 줄일 수는 있지만 없앨 수 없어서 elapsed time은 monotonic clock으로 재고, lease 소유권은 fencing token으로 보호하며 event 순서는 logical clock이나 version을 사용하겠습니다. Token과 certificate 유효 기간처럼 wall clock이 필요한 검증에는 명시적인 허용 skew를 두되 너무 넓혀 보안 기간을 무디게 만들면 안 됩니다. Node별 offset과 drift를 관찰하고 예산을 넘으면 시간 기반 작업을 중단하거나 격리하는 정책도 준비합니다.

## 장점

- NTP 상태를 장애 신호로 활용한다.

## 단점

- last-write-wins가 최신 변경을 덮을 수 있다.

## 주의사항 / 실무 팁

- lease 쓰기에는 fencing token을 포함한다.
