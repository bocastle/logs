# Circuit Breaker Tuning 정리

## 핵심 요약

- 장애 전파를 빠르게 끊을 수 있다.
- 임계값이 예민하면 작은 흔들림에도 회로가 자주 열린다.
- dependency별 정상 실패율과 latency 분포를 먼저 측정한다.

## 개념 설명

Circuit Breaker Tuning은 실패율, 느린 호출 비율, 최소 요청 수, open 유지 시간을 dependency 특성에 맞게 조정하는 일이다.

sliding window 안의 실패율과 slow call rate를 보고 open 여부를 결정하며, traffic이 적은 API는 최소 표본 수를 충분히 둔다.

## 예시

```text
window=100 calls
failure_rate_threshold=50%
slow_call_threshold=800ms
minimum_calls=20
open_duration=30s
```

`minimum_calls`가 낮으면 요청 몇 개만 실패해도 회로가 열려 정상 트래픽을 과하게 막을 수 있다.

## 면접 답변 예시

> Circuit breaker tuning은 dependency의 실패율과 느린 호출 비율을 어느 표본에서 판단해 회로를 열지 정하는 작업입니다. Traffic이 적은 API에서 minimum call 수가 너무 작으면 몇 건의 우연한 실패로 열리고, 너무 크거나 threshold가 둔하면 이미 thread가 포화된 뒤에야 반응합니다. 평상시 실패율과 latency 분포를 먼저 측정하고 open, half-open probe와 fallback 품질을 dependency별로 설정하겠습니다. 장애 중 임계값을 즉흥적으로 바꾸지 않도록 배포 전 검증과 운영 변경 절차도 정합니다.

## 장점

- 느린 dependency로 인한 thread 고갈을 줄인다.

## 단점

- 임계값이 둔하면 이미 포화된 뒤에야 열린다.

## 주의사항 / 실무 팁

- open, half-open, fallback 비율을 대시보드에 둔다.
