# Multi-region Clock Drift 정리

## 핵심 요약

- region 간 시간 비교의 신뢰 범위를 명시한다.
- 큰 drift는 token과 certificate 검증을 실패시킨다.
- region별 NTP offset과 uncertainty를 중앙 수집한다.

## 개념 설명

multi-region clock drift는 region별 host가 서로 다른 NTP source와 network 조건 때문에 wall clock 속도와 offset이 벌어지는 현상이다.

각 region의 offset uncertainty를 측정하고 cross-region timestamp 비교에는 uncertainty interval, logical sequence 또는 hybrid logical clock을 사용한다.

## 예시

```text
Seoul event time=10:00:00.400 uncertainty=±120ms
Virginia event time=10:00:00.300 uncertainty=±180ms
interval overlap -> 실제 순서 단정 불가
```

timestamp 차이가 uncertainty보다 작으면 last-write-wins로 최신성을 단정하지 말고 conflict 또는 causal metadata를 적용한다.

## 면접 답변 예시

> Multi-region clock drift는 region별 host의 wall clock offset이 달라져 timestamp 순서, token 만료와 lease 판단이 흔들리는 문제입니다. Region별 NTP offset과 uncertainty를 수집하고 허용치를 넘은 node는 lease owner나 시간 민감 작업에서 제외하겠습니다. 서로 다른 region의 timestamp 구간이 겹치면 실제 선후를 단정하지 않고 sequence, version이나 hybrid logical clock 같은 causal metadata를 사용합니다. Token leeway는 제한된 drift budget 안에서만 두고 큰 오차를 허용하는 대신 time source 장애를 경보와 application correctness 지표로 연결합니다.

## 장점

- lease와 token TTL에 drift budget을 반영한다.

## 단점

- LWW가 실제 최신 write를 잃을 수 있다.

## 주의사항 / 실무 팁

- 임계 초과 node는 lease owner에서 제외한다.
