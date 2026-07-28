# Redis TTL jitter로 만료 폭주 줄이는 법 정리

## 핵심 요약

- TTL jitter는 비슷한 시각에 저장된 cache key의 만료 시점을 무작위로 흩어 origin 요청이 한꺼번에 몰리는 일을 줄인다.
- Jitter 범위는 최소·최대 TTL이 데이터의 신선도와 갱신 주기를 벗어나지 않게 정한다.
- 하나의 hot key가 만료될 때 생기는 stampede에는 lock이나 stale-while-revalidate 같은 별도 보호가 필요하다.

## 개념 설명

배포나 batch warm-up에서 많은 key를 동시에 저장하면 기본 TTL도 같은 시각에 끝난다. TTL jitter는 각 key의 만료 시간에 작은 무작위 편차를 더해 database와 API로 향하는 miss를 시간상 분산한다.

고정 초 또는 비율 범위를 사용할 수 있다. 음수 jitter를 포함하면 최소 TTL이 지나치게 짧아지지 않는지 확인하고, 양수 범위는 허용 가능한 stale 시간보다 길어지지 않게 한다. 모든 process에서 같은 seed와 순서를 사용해 다시 동기화되지 않는지도 본다.

## 예시

```text
base_ttl=300s
jitter=random(-30s, +30s)
SETEX product:42 ttl=base_ttl+jitter
watch: expired_keys, db_qps_spike
```

이 방식은 여러 key의 동시 만료를 완화하지만 `product:42` 하나에 수많은 요청이 몰리는 문제는 막지 못한다. 그런 key에는 single-flight lock, stale 응답 또는 background refresh를 함께 검토한다.

## 면접 답변 예시

> TTL jitter는 비슷한 시각에 저장된 cache key의 만료 시간을 조금씩 다르게 만들어 origin miss를 분산하는 방법입니다. 기본 TTL에 무작위 범위를 더하되 최소·최대 값이 업무의 신선도 요구를 벗어나지 않게 계산합니다. 이 방법은 여러 key의 만료를 흩어 줄 뿐 하나의 hot key stampede를 막지는 못합니다. 그래서 hot key에는 single-flight나 stale-while-revalidate를 별도로 적용하고 expired key와 origin QPS를 같이 보겠습니다.

## 장점

- Redis server 설정 변경 없이 application의 cache 저장 로직에서 적용하기 쉽다.
- 배포와 warm-up 뒤 다수 key의 동시 만료로 생기는 origin spike를 줄일 수 있다.

## 단점

- Jitter가 너무 크면 key마다 실제 신선도와 갱신 시점이 지나치게 달라진다.
- Hot key 하나의 만료 폭주는 여전히 남을 수 있다.

## 주의사항 / 실무 팁

- `expired_keys`, cache miss와 origin QPS의 시간 분포를 함께 본다.
- Jitter의 최소·최대 TTL과 random seed 동작을 단위 테스트로 고정한다.
