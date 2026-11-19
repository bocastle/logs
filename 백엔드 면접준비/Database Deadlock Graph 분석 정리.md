# Database Deadlock Graph 분석 정리

## 핵심 요약

- cycle을 만든 정확한 resource 순서를 찾는다.
- deadlock log가 짧으면 이전 SQL을 놓친다.
- transaction id와 이전 SQL을 trace에 연결한다.

## 개념 설명

deadlock graph는 transaction을 node, lock wait 의존성을 edge로 표현해 cycle을 이루는 blocker 관계를 보여 주는 wait-for graph다.

DB deadlock detector가 cycle을 찾으면 비용 기준으로 victim transaction 하나를 abort해 나머지 lock을 진행시킨다.

## 예시

```text
T1 holds account:1 -> waits account:2
T2 holds account:2 -> waits account:1
wait-for graph T1 -> T2 -> T1; victim=T2
```

victim SQL 한 줄만 보지 말고 각 transaction이 먼저 획득한 lock과 전체 statement 순서를 재구성해야 원인을 고친다.

## 면접 답변 예시

> Deadlock graph는 transaction을 node, lock wait를 edge로 그려 `T1 -> T2 -> T1` 같은 cycle을 찾는 자료입니다. Victim SQL 한 줄만 보지 않고 각 transaction이 먼저 실행한 statement와 resource 획득 순서를 trace와 DB log로 재구성하겠습니다. DB가 victim 하나를 중단해 cycle을 풀어 주는 것은 증상 처리일 뿐이므로 모든 code path의 lock 순서를 통일하고 transaction 범위를 줄입니다. Victim retry는 작업이 멱등할 때만 jitter와 횟수 제한을 두며 같은 cycle이 반복되는 비율을 별도로 봅니다.

## 장점

- 일관된 lock ordering 규칙을 도출한다.

## 단점

- 재시도만 추가하면 같은 cycle이 반복된다.

## 주의사항 / 실무 팁

- 모든 code path의 resource 획득 순서를 통일한다.
