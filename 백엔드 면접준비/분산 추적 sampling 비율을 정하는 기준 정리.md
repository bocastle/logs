# 분산 추적 sampling 비율을 정하는 기준 정리

## 핵심 요약

- 저장·collector budget을 수치로 관리할 수 있다.
- 요청당 trace 크기를 과소평가하면 저장 비용이 예산을 넘는다.
- route별 요청량과 평균 span 수를 월별로 재계산한다.

## 개념 설명

분산 추적 sampling 비율은 수집 가능한 budget 안에서 평시 트래픽을 대표하고 오류·저지연 trace를 충분히 남기도록 정하는 sampling rate다.

초당 요청 수, 평균 trace 크기, 보존 일수로 저장 budget을 계산한 뒤 목표 traces per second를 정하고, 여기서 기본 sampling rate를 역산한다.

## 예시

```text
incoming: 20_000 requests/s
collector budget: 1_000 traces per second
base sampling rate: 5%
reserve: errors and p99 traces at 100%
```

단순 5%라는 값보다 traces per second 상한과 오류 예약량을 같이 두어야 트래픽 급증 시 collector 포화를 예측할 수 있다.

## 면접 답변 예시

> 오류와 p99 예약량을 두어 평시 샘플이 장애 trace를 밀어내지 않게 한다. 분모인 요청량 변화를 무시하면 고정 sampling rate가 과도하거나 부족해진다. 장애 중 sampling rate를 올릴 때 budget 초과를 막는 우선순위를 정한다.

## 장점

- 트래픽이 다른 route별로 필요한 sampling rate를 정할 수 있다.

## 단점

- 오류 100% 보존 비율을 예약하지 않으면 장애 때 collector가 먼저 포화된다.

## 주의사항 / 실무 팁

- collector에 traces per second 상한과 drop 지표를 둔다.
