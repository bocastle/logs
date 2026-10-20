# API Gateway Circuit Breaking 정리

## 핵심 요약

- 공통 진입점에서 장애 전파를 빠르게 줄인다.
- gateway만 보고 열면 서비스 내부 fallback 기회를 놓칠 수 있다.
- route와 upstream 단위 회로를 분리한다.

## 개념 설명

API Gateway Circuit Breaking은 gateway가 upstream 서비스 상태를 보고 특정 경로 호출을 빠르게 차단하거나 fallback하는 정책이다.

gateway는 upstream별 오류율, timeout, connection failure를 집계하고 회로가 열리면 503이나 cached fallback을 반환한다.

## 예시

```text
route /checkout -> upstream payment
5xx rate > 40% for 1m -> open circuit
return 503 + Retry-After: 10
```

gateway 회로는 upstream에 도달하기 전 차단하므로 내부 서비스 thread와 connection을 보호한다.

## 면접 답변 예시

> Gateway circuit breaker는 upstream의 error와 timeout이 일정 기준을 넘을 때 진입점에서 호출을 빠르게 막아 connection과 thread 고갈을 줄입니다. Route와 upstream을 너무 넓게 묶으면 정상 API까지 함께 차단되므로 dependency 특성에 맞게 회로를 나누겠습니다. Open 상태에서는 503과 적절한 `Retry-After` 또는 검증된 cached fallback을 주고 half-open probe로 회복을 확인합니다. Gateway와 service 내부 circuit이 서로 다른 timeout으로 진동하지 않게 전체 deadline과 상태 지표를 함께 맞춥니다.

## 장점

- upstream별 보호 정책을 중앙에서 운영할 수 있다.

## 단점

- 잘못된 route grouping은 정상 API까지 같이 차단한다.

## 주의사항 / 실무 팁

- gateway circuit 상태를 운영자가 볼 수 있게 노출한다.
