# Cache Warming 정리

## 핵심 요약

- 최초 접속의 miss latency를 줄인다.
- 오래된 hot key 목록은 쓰지 않는 value만 채워 메모리를 낭비한다.
- hot key 목록에 생성 시각과 근거 기간을 포함한다.

## 개념 설명

Cache Warming은 cold cache에서 최초 사용자 요청이 원본 조회 비용을 모두 부담하지 않도록 hot key를 미리 prefetch하는 캐시 초기화 기법이다.

최근 접근 빈도, 상품 노출 계획, 일정된 이벤트에서 hot key 목록을 만들고, 제한된 동시성으로 loader를 실행해 TTL과 함께 prefetch한다.

## 예시

```text
hot key source: top_product_ids(last_24h)
prefetch concurrency=20, rate=500 keys/s
SET product:42 ttl=300s+jitter
watch: cache_hit_ratio, loader_latency, origin_qps
```

Cache Warming은 전체 keyspace를 복사하는 작업이 아니라 hot key 최소 집합으로 cache_hit_ratio를 빠르게 올리는 작업이다.

## 면접 답변 예시

> Cache warming은 배포나 cache 초기화 직후 첫 사용자들이 miss 비용을 전부 부담하지 않도록 예상 hot key를 미리 채우는 방식입니다. 전체 keyspace를 복사하기보다 최근 사용량과 예정된 노출을 근거로 최소 집합을 만들고 제한된 동시성으로 읽겠습니다. 오래된 목록이나 곧 변경될 데이터를 채우면 memory만 쓰고 stale 값이 될 수 있어 목록 생성 시각과 TTL도 함께 관리해야 합니다. Warming 전후의 hit ratio, origin QPS와 loader latency를 비교해 대상 수와 속도를 조정합니다.

## 장점

- hot key 폭주로부터 DB와 외부 API를 보호한다.

## 단점

- prefetch 동시성이 높으면 원본을 직접 포화시킨다.

## 주의사항 / 실무 팁

- prefetch loader에 timeout, retry 상한, concurrency 제한을 둔다.
