# Cache TTL Jitter 정리

## 핵심 요약

- 동시 만료로 인한 origin QPS spike를 줄인다.
- jitter가 너무 크면 데이터 신선도 기준이 흐려진다.
- 허용 stale 시간 안에서 jitter 범위를 정한다.

## 개념 설명

Cache TTL Jitter는 캐시 항목의 만료 시간이 한꺼번에 겹치지 않도록 TTL에 작은 무작위 변동을 주는 일반 캐시 전략이다.

base TTL에 난수 범위를 더해 key별 expiration을 흩뜨리고, 원본 보호가 필요한 key에는 lock이나 refresh-ahead를 같이 둔다.

## 예시

```text
base=10m, jitter=0..90s
expires_at = now + base + random(jitter)
watch: expiration_spike, origin_qps
```

jitter는 만료 폭주를 낮추는 확률적 완화다. 원본이 매우 약하면 stampede lock 같은 강한 제어가 더 필요하다.

## 면접 답변 예시

> Cache TTL jitter는 같은 시점에 채워진 cache key들이 한꺼번에 만료돼 origin으로 요청이 몰리는 현상을 줄이는 방법입니다. 기본 TTL에 허용 가능한 범위의 난수를 더하거나 빼서 만료 시점을 분산하되, 데이터 신선도 기준을 넘지 않도록 범위를 정하겠습니다. 다만 사용량이 큰 hot key 하나가 만료되는 문제까지 해결하지는 못합니다. 그런 key에는 single-flight lock이나 background refresh를 같이 적용하고 cache miss burst와 origin QPS로 효과를 확인합니다.

## 장점

- 애플리케이션 레벨에서 쉽게 적용할 수 있다.

## 단점

- hot key 단일 만료에는 효과가 제한적이다.

## 주의사항 / 실무 팁

- origin QPS와 cache miss burst를 함께 본다.
