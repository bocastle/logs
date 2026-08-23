# Retry Jitter 정리

## 핵심 요약

- 재시도 동시 폭주를 줄인다.
- 최대 시도 제한이 없으면 jitter가 있어도 부하가 계속된다.
- max attempts와 overall deadline을 같이 둔다.

## 개념 설명

Retry Jitter는 여러 클라이언트가 같은 간격으로 재시도해 다시 폭주하는 일을 막기 위해 backoff 시간에 무작위성을 넣는 기법이다.

exponential backoff에 full jitter 또는 decorrelated jitter를 적용하고, Retry-After가 있으면 서버 지시를 우선한다.

## 예시

```text
base=100ms, cap=2s
sleep = random(0, min(cap, base * 2^attempt))
```

full jitter는 같은 attempt의 클라이언트라도 sleep 시간이 흩어져 upstream 회복 시간을 벌어 준다.

## 면접 답변 예시

> Retry jitter는 장애 직후 여러 client가 같은 간격으로 재시도해 upstream을 다시 몰아붙이는 현상을 줄이는 방법입니다. Exponential backoff에 full jitter 같은 무작위 지연을 적용하고, server가 `Retry-After`를 보냈다면 그 지시를 우선하겠습니다. Jitter가 있어도 무한 재시도는 막지 못하므로 max attempts와 전체 deadline을 함께 둬야 합니다. 특히 쓰기 요청은 idempotency key 없이 재시도하면 중복 부작용이 생길 수 있어 재시도 가능 조건과 최종 실패 사유까지 metric으로 남깁니다.

## 장점

- 일시 장애 회복 중 서버에 숨 쉴 시간을 준다.

## 단점

- Retry-After를 무시하면 rate limit과 충돌한다.

## 주의사항 / 실무 팁

- 429와 503은 Retry-After를 우선한다.
