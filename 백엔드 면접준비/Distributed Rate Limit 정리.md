# Distributed Rate Limit 정리

## 핵심 요약

- 노드가 늘어도 tenant별 한도를 일관되게 지킬 수 있다.
- 중앙 limiter 지연이 모든 요청 지연에 더해진다.
- 로컬 캐시 한도와 중앙 한도의 오차 허용치를 정한다.

## 개념 설명

Distributed Rate Limit은 여러 API 서버나 gateway 인스턴스가 같은 한도 상태를 공유하는 방식이다.

중앙 저장소나 전용 limiter 서비스에서 토큰을 원자적으로 차감하고, 장애 시에는 fail-open 또는 fail-closed 정책을 명확히 정한다.

## 예시

```text
gateway -> limiter.reserve(key, cost)
allowed: forward
denied: 429 Retry-After
limiter down: local emergency limit
```

limiter 장애 정책을 두면 한도 시스템 문제가 전체 API 장애로 번지는 범위를 정할 수 있다.

## 면접 답변 예시

> Distributed rate limit은 여러 gateway instance가 tenant나 client의 같은 한도 상태를 공유하게 하는 설계입니다. 중앙 limiter에서 token을 원자적으로 차감하면 정확도는 높지만 그 latency와 장애가 모든 request 경로에 들어옵니다. 지역별 token allocation이나 local cache를 쓰면 처리량은 좋아지는 대신 허용 오차만큼 초과 사용이 생길 수 있어 error budget을 명시하겠습니다. Limiter 장애 때 결제처럼 보호할 API는 fail-closed, 저위험 조회는 emergency local limit처럼 위험도별 정책을 나누고 decision과 denied reason을 감사 가능하게 남깁니다.

## 장점

- 전역 quota와 지역별 quota를 함께 운영할 수 있다.

## 단점

- 저장소 장애 때 정책이 없으면 전체 API가 흔들린다.

## 주의사항 / 실무 팁

- limiter latency와 denied count를 핵심 지표로 둔다.
