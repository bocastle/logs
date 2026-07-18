# Slow query fingerprint로 병목을 묶어 보는 법 정리

## 핵심 요약

- Query fingerprint는 literal과 bind 값을 제거해 같은 SQL 형태의 실행을 하나로 묶는 식별자다.
- 호출 수, 누적 시간, 평균과 tail latency, 처리 행 수를 함께 봐야 최적화 우선순위를 정할 수 있다.
- 같은 fingerprint도 bind 값의 분포와 실행 계획이 다를 수 있어 느린 표본의 plan과 데이터 skew를 별도로 확인한다.

## 개념 설명

Slow query fingerprint는 SQL의 상수와 bind 값을 일반화한 normalized SQL을 hash해 같은 쿼리군을 묶는다. 고객 ID만 다른 수천 개의 SQL을 한 작업으로 집계해 전체 부하에서 차지하는 비중을 볼 수 있다.

데이터베이스 통계에서 호출 수와 누적 실행 시간을 보고, 필요하면 APM이나 query log 집계로 p95 같은 분포를 보완한다. 도구마다 정규화 규칙과 제공 통계가 다르므로 “fingerprint면 p95가 나온다”고 가정하면 안 된다. 평균이 짧아도 호출량이 많아 누적 부하가 큰 쿼리와 드물지만 매우 느린 쿼리를 나눠 본다.

## 예시

```text
SELECT * FROM orders WHERE customer_id = ? ORDER BY created_at DESC LIMIT ?
fingerprint=8f31 calls=120k p95=84ms total=9,800s
```

같은 형태라도 특정 고객의 데이터가 훨씬 많으면 plan과 지연이 달라질 수 있다. 민감한 bind 원문을 그대로 저장하지 말고 값의 범위나 bucket, plan hash처럼 원인 분석에 필요한 최소 정보만 남긴다.

## 면접 답변 예시

> Query fingerprint는 상수와 bind 값을 일반화해 같은 SQL 형태를 하나로 묶는 식별자입니다. 호출 수와 누적 시간으로 전체 부하를 보고, 평균만으로 놓치는 tail latency는 APM이나 query log 분포로 보완하겠습니다. 같은 fingerprint라도 bind 분포와 plan이 다를 수 있어 느린 구간의 plan hash와 row estimate를 비교해야 합니다. Bind 원문은 민감정보가 될 수 있으므로 제한된 표본만 안전하게 수집합니다.

## 장점

- 배포 전후 호출 수와 누적 시간, 지연 분포를 같은 쿼리 형태로 비교할 수 있다.
- Literal이 다른 SQL을 하나로 묶어 실제 누적 부하가 큰 작업을 찾기 쉽다.

## 단점

- 과도한 정규화는 데이터 분포와 plan이 다른 실행을 같은 문제로 합칠 수 있다.
- 평균만 보면 드문 tail latency와 특정 bind 구간의 병목을 숨긴다.

## 주의사항 / 실무 팁

- Row estimate와 실제 row 수, plan hash가 크게 다른 bind 범위를 분리해 본다.
- 상위 fingerprint에만 제한적으로 `EXPLAIN`을 수집하고 bind와 SQL의 개인정보를 마스킹한다.
