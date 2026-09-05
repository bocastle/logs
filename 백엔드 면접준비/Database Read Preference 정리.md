# Database Read Preference 정리

## 핵심 요약

- 읽기 부하를 replica로 분산한다.
- secondary는 replication lag만큼 오래된 값을 반환한다.
- 업무별 consistency 요구에 따라 mode를 나눈다.

## 개념 설명

read preference는 replica set에서 읽기를 primary, secondary 또는 지연·지역 조건에 따라 어느 node로 보낼지 정하는 routing 정책이다.

primaryPreferred는 primary가 가능하면 그곳을 읽고 장애 때 secondary로 fallback하며 max staleness와 tag를 함께 쓰면 오래된 원격 replica를 제외한다.

## 예시

```text
mode=primaryPreferred
maxStalenessSeconds=30
tagSets=[{region: seoul}]
```

강한 일관성이 필요한 잔액 조회에 secondaryPreferred를 쓰면 failover가 없어도 staleness가 보일 수 있다.

## 면접 답변 예시

> MongoDB read preference는 replica set에서 read를 primary와 secondary 중 어디로 보낼지 정하는 routing 정책입니다. Secondary로 부하와 지역 latency를 줄일 수 있지만 replication lag만큼 오래된 값을 받을 수 있고 fallback mode에서는 요청마다 consistency 의미가 달라질 수 있습니다. 잔액이나 방금 쓴 값 확인처럼 최신성이 필요한 경로는 primary를 사용하고, 허용 가능한 조회만 tag와 max staleness 조건으로 secondary에 보내겠습니다. 실제 선택 node와 지연 상태를 관찰해 가용성 이득이 업무 정합성보다 큰지 확인합니다.

## 장점

- 지역 tag로 가까운 node를 선택한다.

## 단점

- fallback 때 일관성 의미가 요청마다 바뀔 수 있다.

## 주의사항 / 실무 팁

- read-your-writes 경로는 primary 읽기나 요구 조건에 맞는 causal-consistent session을 검토한다.
