# API Throttling과 Queuing 정리

## 핵심 요약

- 짧은 트래픽 스파이크를 흡수할 수 있다.
- 대기열이 길면 timeout 직전 요청만 쌓인다.
- 사용자별 429와 전역 queue 503 비율을 분리해 본다.

## 개념 설명

API Throttling은 요청 속도를 늦추는 정책이고, Queuing은 처리 가능한 시점까지 요청을 대기시키는 완충 장치다.

사용자별·tenant별 rate limit 초과는 429로 알리고, 전역 queue 포화나 서버 처리 용량 부족은 503으로 구분해 tail latency 폭주를 막는다.

## 예시

```text
if user_rate_exceeded:
  return 429 Too Many Requests + Retry-After
elif global_queue_depth > 200 or wait_ms > 100:
  return 503 Service Unavailable + Retry-After
else:
  enqueue request
```

429는 특정 호출자의 요청 속도 계약 위반이고, 503은 서비스 전체가 현재 요청을 처리하기 어렵다는 뜻이다. queue limit 없이 모두 기다리게 하면 사용자 지연과 upstream 압박이 더 커진다.

## 면접 답변 예시

> API throttling은 caller의 요청 속도를 제한하고 queuing은 짧은 spike를 처리 가능 시점까지 완충하는 방식입니다. 특정 client나 tenant의 계약 초과는 429, service 전체 queue 포화는 503처럼 원인과 retry 의미를 구분하겠습니다. Queue가 request deadline 가까이까지 길어지면 처리할 수 없는 요청만 쌓이므로 depth와 예상 wait에 상한을 두고 일찍 거절해야 합니다. `Retry-After`와 jittered backoff를 문서화하고 긴 작업은 동기 queue 대신 job ID를 반환하는 흐름을 검토합니다.

## 장점

- 서버 처리량을 안정적인 범위에 묶는다.

## 단점

- 우선순위가 없으면 중요한 요청도 뒤에 묶인다.

## 주의사항 / 실무 팁

- 긴 작업은 동기 대기 대신 job id 반환을 검토한다.
