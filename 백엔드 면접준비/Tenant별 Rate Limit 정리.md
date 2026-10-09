# Tenant별 Rate Limit 정리

## 핵심 요약

- 요금제별 처리량 계약을 일관되게 적용할 수 있다.
- plan 정보 갱신이 느리면 요금제 변경 후에도 오래된 quota가 적용된다.
- plan quota 변경은 버전과 적용 시각을 감사 로그에 남긴다.

## 개념 설명

Tenant별 Rate Limit은 tenant의 plan quota와 전체 서비스 용량을 동시에 반영해 허용·차단을 결정하는 다계층 정책이다.

요청은 tenant별 plan quota와 route cost, 서비스 전체 capacity 보호를 차례로 통과해야 한다. tenant 계약 초과는 429로, 전체 서비스가 일시적으로 처리할 수 없는 상태는 503처럼 원인이 드러나는 응답으로 구분한다.

## 예시

```text
plan quota: PRO=1200 requests/min
route cost: export=20, read=1
global limit: 50_000 requests/min
denied -> 429 reason=TENANT_PLAN_LIMIT
global capacity exhausted -> 503 reason=SERVICE_CAPACITY
```

plan quota만 있고 global limit이 없으면 많은 tenant가 동시에 한도를 쓸 때 서비스 전체를 보호하지 못한다.

## 면접 답변 예시

> Tenant rate limit은 plan quota와 route별 비용을 tenant 단위로 집행하면서 별도의 global capacity guard로 서비스 전체도 보호하는 다계층 정책입니다. Tenant 계약을 넘으면 429와 reset 정보를 주고, 여러 tenant가 동시에 몰려 전체 capacity가 부족하면 503으로 일시 장애 의미를 구분하겠습니다. Route cost는 caller가 예측할 수 있게 문서화하고 plan 변경은 version과 적용 시각을 남깁니다. Tenant별 429 비율과 global capacity rejection을 분리해 어느 계층이 실제 병목인지 관찰합니다.

## 장점

- 비싼 route에 가중치를 줘 실제 자원 사용을 더 잘 반영한다.

## 단점

- global limit이 너무 낮으면 서로 다른 tenant이 같이 429를 받는다.

## 주의사항 / 실무 팁

- tenant 429 비율과 global limit 사용률을 분리해 본다.
