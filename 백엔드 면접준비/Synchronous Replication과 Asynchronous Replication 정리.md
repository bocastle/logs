# Synchronous Replication과 Asynchronous Replication 정리

## 핵심 요약

- Synchronous replication은 정해진 standby acknowledgement를 기다린 뒤 commit 성공을 반환해 failover의 데이터 손실 창을 줄인다.
- Asynchronous replication은 primary local commit 뒤 바로 응답해 latency와 가용성이 좋지만 전송 전 장애의 RPO를 남긴다.
- 업무별 RPO·RTO와 latency를 기준으로 quorum, remote 지역과 장애 시 policy를 정한다.

## 개념 설명

동기 복제에서는 primary가 local WAL을 기록한 뒤 설정된 standby의 write·flush 또는 apply acknowledgement를 기다릴 수 있다. 어느 단계까지 기다리는지는 database 설정에 따라 실제 보장과 latency가 달라진다. 비동기 복제는 local commit 뒤 client에 응답하고 standby가 나중에 따라온다.

동기 standby가 느리거나 끊기면 commit latency가 늘거나 쓰기가 멈출 수 있다. Quorum과 후보 우선순위를 잘못 설정하면 “동기”라는 이름과 기대한 장애 내구성이 달라진다. 비동기는 remote region 지연을 쓰기 경로에서 빼지만 failover 때 아직 전송되지 않은 transaction을 잃을 수 있다.

## 예시

```text
sync: primary WAL flush -> standby flush acknowledgement -> commit success
async: primary WAL flush -> commit success -> standby replay later
```

결제 원장처럼 낮은 RPO가 필요한 데이터와 재생성 가능한 projection을 같은 정책으로 묶지 않는다. Failover drill에서 설정상 보장이 아니라 실제 commit latency와 손실 범위를 측정한다.

## 면접 답변 예시

> Synchronous replication은 설정된 standby의 acknowledgement를 기다린 뒤 commit 성공을 반환해 failover의 데이터 손실 창을 줄입니다. 그 대신 standby 지연과 장애가 primary write latency와 가용성에 영향을 줍니다. Asynchronous 방식은 빠르지만 replication lag만큼 손실 가능성이 남습니다. 업무별 RPO와 latency를 기준으로 적용하고 quorum·ack 단계와 standby 장애 시 policy를 failover test로 검증하겠습니다.

## 장점

- 동기 복제는 failover 시 확인된 commit의 유실 가능성을 낮춘다.
- 비동기 복제는 remote region 지연을 write path에서 빼 latency와 가용성을 높인다.

## 단점

- 동기 standby 장애와 지연이 primary commit을 멈추거나 느리게 할 수 있다.
- 비동기 failover는 아직 전송되지 않은 transaction을 잃을 수 있다.

## 주의사항 / 실무 팁

- 동기 후보 장애 시 quorum을 바꿀 조건과 승인 절차를 미리 준비한다.
- Commit p99, standby acknowledgement 단계와 실제 failover 손실을 정기적으로 측정한다.
