# Blue-Green Deployment 정리

## 핵심 요약

- 배포와 rollback이 빠르다.
- 두 환경 비용이 동시에 든다.
- 전환 전 green warm-up과 smoke test를 자동화한다.

## 개념 설명

Blue-Green Deployment는 현재 운영 환경(blue)과 새 환경(green)을 동시에 준비한 뒤 traffic switch로 배포하는 방식이다.

green에 새 버전을 배포하고 smoke test와 health check를 통과하면 load balancer나 gateway routing을 green으로 전환한다.

## 예시

```text
blue: v1 receiving traffic
green: v2 warmed and healthy
switch 100% traffic to green
rollback: route back to blue
```

blue를 즉시 제거하지 않으면 전환 직후 문제가 생겼을 때 route만 되돌려 빠르게 복구할 수 있다.

## 면접 답변 예시

> Blue-Green deployment는 현재 blue와 새 green 환경을 동시에 준비한 뒤 routing만 바꿔 배포하는 방식입니다. Green을 충분히 warm-up하고 smoke test한 뒤 전환하면 문제가 생겼을 때 route를 blue로 빠르게 되돌릴 수 있습니다. 하지만 두 버전이 함께 접근하는 DB schema는 backward compatible해야 하고, session이나 cache가 환경별로 갈리지 않게 설계해야 실제 rollback이 가능합니다. 전환 직후에는 error rate와 latency를 짧은 간격으로 확인하고 안정화 전까지 blue를 보존하겠습니다.

## 장점

- 새 환경을 실제와 비슷한 상태로 미리 검증할 수 있다.

## 단점

- DB schema가 backward compatible하지 않으면 되돌림이 어렵다.

## 주의사항 / 실무 팁

- DB 변경은 expand-contract로 양쪽 버전이 읽게 만든다.
