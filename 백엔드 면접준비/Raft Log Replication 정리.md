# Raft Log Replication 정리

## 핵심 요약

- 복제 log의 단일 순서를 합의한다.
- 느린 quorum은 commit latency를 높인다.
- term·matchIndex·commit index를 관찰한다.

## 개념 설명

Raft log replication은 leader가 명령을 index·term 순서로 log에 추가하고 quorum follower가 복제한 entry만 commit하는 합의 절차다.

AppendEntries의 prevLogIndex와 prevLogTerm으로 prefix 일치를 확인하고 불일치 follower의 nextIndex를 되돌려 leader log와 맞춘다.

## 예시

```text
AppendEntries(term=8, prevLogIndex=41, prevLogTerm=7, entries=[42], leaderCommit=41)
quorum ack for index 42 -> commit index=42
```

leader는 현재 term의 entry가 quorum에 저장된 뒤 commit해야 이전 term entry까지 안전하게 commit됐다고 판단할 수 있다.

## 면접 답변 예시

> Raft에서는 leader가 log entry를 follower에 복제하고 quorum에 저장된 안전한 entry만 commit해 모든 state machine이 같은 순서로 적용하게 합니다. `prevLogIndex`와 `prevLogTerm`이 맞지 않으면 follower의 다음 복제 위치를 되돌려 공통 prefix부터 다시 맞춥니다. Leader는 현재 term의 entry가 quorum에 복제된 것을 기준으로 commit index를 전진시켜 이전 term entry를 섣불리 commit하는 문제를 피합니다. Membership은 두 독립 quorum이 생기지 않도록 joint consensus 같은 합의된 변경 절차로 처리하겠습니다.

## 장점

- follower 불일치를 자동으로 되감아 복구한다.

## 단점

- 큰 log gap은 follower catch-up을 오래 끈다.

## 주의사항 / 실무 팁

- snapshot으로 오래된 log를 압축한다.
