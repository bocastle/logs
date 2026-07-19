# Read replica에서 read-your-writes를 보장하는 법 정리

## 핵심 요약

- Read-your-writes는 한 세션이 방금 쓴 값을 이후 자신의 읽기에서 다시 볼 수 있게 하는 일관성 보장이다.
- 쓰기 위치 token과 replica의 replay 위치를 비교하거나 짧은 primary stickiness로 replication lag 구간을 피한다.
- 모든 읽기를 primary로 보내지 말고 쓰기 직후처럼 보장이 필요한 흐름에만 비용을 지불한다.

## 개념 설명

비동기 read replica는 primary의 변경을 약간 늦게 적용한다. 사용자가 저장 직후 상세 화면을 replica에서 읽으면 이전 값이 보여 저장이 취소된 것처럼 느낄 수 있다. Read-your-writes는 적어도 같은 사용자나 세션의 후속 읽기에서 자신의 쓰기를 보게 하는 보장이다.

데이터베이스가 제공하는 commit 위치 token을 쓰기 응답이나 세션에 저장하고, replica가 그 위치까지 replay했는지 확인한 뒤 읽을 수 있다. 짧게 기다렸는데 따라오지 못하면 primary로 fallback한다. 제품에 종속된 위치 token을 사용하기 어렵다면 일정 시간 primary에 고정하는 단순한 방법도 있지만 실제 lag 변동을 정확히 반영하지 못한다.

## 예시

```text
commit_lsn=16/B374D848
replica replay_lsn=16/B374D120 -> primary로 읽기
replica replay_lsn>=commit_lsn -> replica 허용
```

고정 sleep 대신 실제 복제 진척도를 비교한다. Replica 대기를 무제한 허용하면 일관성은 지켜도 응답 SLO를 잃으므로 짧은 상한과 primary fallback을 둔다.

## 면접 답변 예시

> Read-your-writes는 사용자가 방금 쓴 값을 자신의 다음 읽기에서 볼 수 있게 하는 세션 일관성입니다. 쓰기 시점의 commit LSN이나 version token을 보관하고, replica가 그 위치까지 적용한 경우에만 읽게 할 수 있습니다. 짧은 시간 안에 따라오지 못하면 primary로 fallback하고, 보장이 필요 없는 백그라운드 조회는 그대로 replica를 사용합니다. 모든 읽기를 primary에 고정하지 않고 저장 직후 흐름에만 비용을 지불하겠습니다.

## 장점

- 일관성 보장이 필요 없는 요청은 replica로 보내 primary 읽기 부하를 계속 분산할 수 있다.
- 저장 직후 이전 값이 보이는 사용자 혼란을 줄인다.

## 단점

- LSN 같은 위치 token의 획득과 비교 방식은 데이터베이스 제품에 종속된다.
- 세션 token이 유실되거나 다른 기기로 이어지면 보장 범위가 끊길 수 있다.

## 주의사항 / 실무 팁

- Replica 대기에는 짧은 상한과 primary fallback을 둬 응답 SLO를 보호한다.
- 저장 직후 조회, 세션 token 유실, replica 장기 지연을 통합 테스트한다.
