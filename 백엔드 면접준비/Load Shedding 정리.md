# Load Shedding 정리

## 핵심 요약

- Load shedding은 시스템이 완전히 포화되기 전에 일부 요청을 빠르게 거절해 핵심 경로의 성공률을 보호한다.
- CPU 하나만 보지 않고 queue wait, in-flight 수, tail latency와 dependency 상태를 결합해 시작·종료 조건을 정한다.
- 사용자별 rate limit의 429와 서비스 전체 과부하의 503을 구분하고 재시도 폭주를 막는다.

## 개념 설명

시스템이 포화되면 모든 요청이 조금씩 느려지는 데서 끝나지 않고 thread, connection과 queue가 고갈돼 전체 timeout으로 번질 수 있다. Load shedding은 성공 가능성이 낮거나 우선순위가 낮은 요청을 입구에서 거절해 남은 용량을 핵심 기능에 사용한다.

시작 신호는 CPU, queue wait, in-flight 요청과 latency를 조합하고 종료에는 낮은 별도 threshold와 안정 시간을 둬 상태가 자주 흔들리지 않게 한다. 특정 tenant가 계약 한도를 넘긴 경우는 429, 서비스 전체가 처리할 수 없는 경우는 503처럼 의미를 나눈다.

## 예시

```text
if cpu > 85% and queue_depth > 500:
  reject low priority search with 503
  keep checkout and login
```

로그인과 결제를 무조건 보호한다는 고정 규칙보다 각 API의 business 우선순위와 dependency 비용을 사전에 합의한다. 거절 응답에는 client가 안전하게 retry할 수 있는지와 `Retry-After` 정책을 문서화한다.

## 면접 답변 예시

> Load shedding은 시스템이 완전히 포화되기 전에 일부 요청을 빠르게 거절해 핵심 API의 성공률을 보호하는 전략입니다. Queue wait, in-flight 수와 tail latency를 보고 우선순위가 낮거나 비용이 큰 요청부터 제한하겠습니다. 사용자 계약 초과는 429, 서비스 전체 과부하는 503으로 구분하고 retry 가능 여부와 backoff를 알립니다. 시작과 종료 threshold에 hysteresis를 두고 실제 거절이 dependency 부하를 줄였는지 확인합니다.

## 장점

- 장애 중에도 핵심 API의 성공률과 응답 시간을 더 오래 유지할 수 있다.
- Timeout 직전까지 자원을 점유할 요청을 입구에서 줄일 수 있다.

## 단점

- 너무 늦게 시작하면 이미 thread와 connection이 고갈돼 거절 처리조차 느려진다.
- 잘못된 우선순위는 중요 요청을 차단하거나 특정 고객에게 불공정하게 작동할 수 있다.

## 주의사항 / 실무 팁

- Shedding 시작과 종료 신호에 hysteresis와 최소 유지 시간을 둔다.
- 거절률, 핵심 API 성공률, queue wait와 client retry 증가를 함께 관찰한다.
