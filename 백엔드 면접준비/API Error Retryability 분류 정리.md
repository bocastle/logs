# API Error Retryability 분류 정리

## 핵심 요약

- 클라이언트와 서버의 retry 폭주를 줄인다.
- 모든 5xx를 재시도하면 이미 포화된 서버를 더 누른다.
- 오류 카탈로그에 retryable 값을 포함한다.

## 개념 설명

API Error Retryability 분류는 어떤 실패가 안전한 재시도 대상인지 API가 명확히 알려 주는 기준이다.

네트워크 timeout, 429, 일부 5xx는 조건부 재시도 가능하지만, 4xx 검증 오류와 중복 부작용이 있는 요청은 idempotency key가 필요하다.

## 예시

```text
408/429/503 -> retry only when request is replay-safe, with backoff
400/403 -> do not retry unchanged request
401 -> refresh credential once when supported, then retry once
409 + idempotency key -> check status then retry
```

오류별 재시도 기준이 있어야 SDK가 같은 요청을 무작정 반복해 장애를 키우지 않는다.

## 면접 답변 예시

> Retryability는 status code 하나가 아니라 요청이 replay-safe한지, server가 처리했을 가능성과 같은 request를 반복해도 부작용이 없는지를 함께 봐야 합니다. 429와 503은 `Retry-After`, backoff와 jitter를 따르고 body를 다시 보낼 수 없는 stream이나 idempotency key 없는 write는 자동 재시도하지 않겠습니다. 401은 지원되는 credential refresh를 한 번 수행한 뒤 한 번만 재시도하고 400·403은 같은 요청을 반복하지 않습니다. SDK에는 전체 deadline과 max attempts를 두고 최종 원인을 보존합니다.

## 장점

- 결제 같은 부작용 API에서 중복 처리를 줄인다.

## 단점

- POST 재시도에 idempotency key가 없으면 중복 생성이 생긴다.

## 주의사항 / 실무 팁

- 쓰기 API는 idempotency key 지원 여부를 문서화한다.
