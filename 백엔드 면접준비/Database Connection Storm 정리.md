# Database Connection Storm 정리

## 핵심 요약

- DB 회복 구간의 인증 CPU를 보호한다.
- 동기 retry는 DB가 회복하기 전에 다시 과부하를 만든다.
- connection create에 concurrency limit을 둔다.

## 개념 설명

connection storm은 DB 재시작이나 network 회복 직후 많은 application instance가 동시에 새 connection을 생성하는 thundering herd 현상이다.

각 pool의 minimum idle refill과 retry가 동기화되면 authentication, TLS, process fork가 몰리므로 exponential backoff, jitter, 전역 connection budget으로 속도를 제한한다.

## 예시

```text
200 pods x minimumIdle 10 = 2000 reconnect attempts
backoff=200ms..10s + jitter, create concurrency=2/pod
DB max_connections=600
```

평상시 pool 크기가 적절해도 cold start 동시성은 별도 peak이므로 deployment와 failover에서 connection create rate를 검증한다.

## 면접 답변 예시

> Database connection storm은 DB나 network가 회복된 직후 많은 instance가 pool을 동시에 채우며 인증, TLS와 connection slot을 다시 고갈시키는 현상입니다. DB `max_connections`를 service별 budget으로 나누고 instance pool 크기뿐 아니라 전체 pod 수와 배포 중 중복 instance까지 계산하겠습니다. Connection 생성에는 concurrency limit, exponential backoff와 full jitter를 적용하고 health check가 업무 traffic보다 먼저 slot을 차지하지 않게 별도 제한을 둡니다. Rolling start와 DB restart 부하 테스트에서 create rate, authentication CPU, active connection과 정상 query 회복 시간을 함께 측정합니다.

## 장점

- instance별 재연결을 시간에 분산한다.

## 단점

- 큰 minimum idle은 사용하지 않을 connection까지 생성한다.

## 주의사항 / 실무 팁

- backoff와 full jitter를 적용한다.
