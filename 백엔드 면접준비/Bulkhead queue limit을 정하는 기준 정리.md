# Bulkhead queue limit을 정하는 기준 정리

## 핵심 요약

- Bulkhead queue는 특정 dependency의 대기 작업 수를 제한해 다른 작업군의 실행 자원을 보호한다.
- 크기는 메모리 여유보다 요청의 남은 timeout과 목표 대기 시간으로 정한다.
- queue depth만 보지 말고 실제 대기 시간, 거절률, dependency 지연을 함께 관찰한다.

## 개념 설명

Bulkhead는 dependency나 작업군마다 동시 실행 수와 대기열을 나눠 한 영역의 지연이 전체 실행 자원을 소모하지 않게 한다. Queue limit은 그 격리 구역 안에서 기다릴 수 있는 작업 수의 상한이다.

대략적인 처리율은 worker 수를 처리 시간으로 나눠 계산할 수 있지만 평균값만 믿으면 tail latency를 놓친다. 보수적인 지연값과 요청의 전체 timeout에서 이미 사용한 시간을 기준으로 대기 한도를 잡고, 그 시간을 넘길 요청은 queue에 넣기 전에 빠르게 거절하는 편이 낫다.

## 예시

```text
workers=20
target_wait<=200ms
avg_latency=50ms
queue_limit ~= workers * target_wait / avg_latency = 80
```

이 계산은 시작점일 뿐이다. 실제 처리 시간 분포가 길게 늘어지면 80개 요청은 200ms보다 훨씬 오래 기다릴 수 있으므로 부하 테스트에서 queue wait p95와 timeout 비율을 확인해 낮춰야 한다.

## 면접 답변 예시

> Bulkhead queue limit은 느린 dependency에 기다리는 요청이 전체 스레드나 연결을 차지하지 않도록 대기열 상한을 두는 값입니다. Worker 수와 처리 시간으로 시작값을 계산하되, 평균보다 보수적인 지연과 요청의 남은 timeout을 기준으로 잡겠습니다. 한도를 넘으면 timeout 직전까지 기다리게 하지 않고 빠르게 거절하거나 fallback으로 전환합니다. 운영에서는 queue depth보다 실제 대기 시간, 거절률, dependency 지연을 함께 보며 조정합니다.

## 장점

- 느린 dependency가 다른 작업군의 thread와 connection까지 고갈시키는 범위를 줄인다.
- 성공 가능성이 낮은 요청을 오래 대기시키기 전에 명확한 거절 신호를 줄 수 있다.

## 단점

- queue가 너무 작으면 정상적인 짧은 스파이크도 과도하게 실패한다.
- 너무 크면 이미 timeout 예산을 잃은 요청이 쌓여 메모리와 처리량만 사용한다.

## 주의사항 / 실무 팁

- 거절 응답에는 재시도 가능 여부와 backoff 기준을 명확히 표시한다.
- bulkhead별 worker 수, queue wait p95, 거절률을 같은 대시보드에서 본다.
