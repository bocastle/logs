# Connection Pool Backpressure 정리

## 핵심 요약

- downstream 포화가 전체 서버 장애로 번지는 것을 줄인다.
- 거절 기준이 낮으면 정상 스파이크를 처리하지 못한다.
- pool_pending과 request latency를 함께 임계값으로 둔다.

## 개념 설명

Connection Pool Backpressure는 connection 대기가 길어질 때 새 요청을 늦추거나 거절해 downstream과 애플리케이션을 보호하는 흐름이다.

pending queue, acquire latency, timeout 비율을 기준으로 admission control을 걸고 서비스 capacity 부족은 503으로 빠르게 반환한다. 특정 caller의 계약 속도 초과를 제한하는 경우에만 429를 사용한다.

## 예시

```text
if pool_pending > 100 or acquire_p95 > 200ms:
  reject low priority requests
  return 503 SERVICE_BUSY
```

connection을 기다리는 요청이 무한히 쌓이면 pool 문제가 API 전체 thread 고갈로 커진다.

## 면접 답변 예시

> Connection pool backpressure는 connection 대기가 길어질 때 새 요청을 admission 단계에서 제한해 downstream 포화가 전체 thread 고갈로 번지는 것을 막습니다. `pool_pending`, acquire latency와 timeout을 보되 CPU와 request queue 같은 전체 포화 지표도 함께 확인하겠습니다. Service capacity 때문에 거절할 때는 503으로 빠르게 실패시키고 caller별 rate 계약 초과의 429와 의미를 섞지 않습니다. 중요 route는 별도 budget을 두고 retry에는 `Retry-After`, backoff와 jitter를 안내합니다.

## 장점

- 대기열 폭주 전에 호출자에게 빠른 실패를 준다.

## 단점

- 거절 응답에 backoff 정보가 없으면 즉시 재시도가 몰린다.

## 주의사항 / 실무 팁

- 중요 API와 낮은 우선순위 API의 admission 기준을 나눈다.
