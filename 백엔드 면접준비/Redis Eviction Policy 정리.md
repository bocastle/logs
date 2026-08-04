# Redis Eviction Policy 정리

## 핵심 요약

- Redis eviction policy는 `maxmemory`에 도달했을 때 어떤 key를 제거하거나 새 쓰기를 거절할지 정한다.
- 캐시라면 전체 key 대상 LRU·LFU 등을 검토하고, 유실되면 안 되는 상태는 별도 instance와 `noeviction` 정책으로 분리한다.
- `evicted_keys` 증가는 정상 무효화가 아니라 memory pressure 신호이므로 miss와 origin 부하를 함께 본다.

## 개념 설명

Redis가 `maxmemory`에 도달하면 설정된 policy에 따라 key를 제거하거나 memory를 늘리는 command를 거절한다. `allkeys-*`는 모든 key를 후보로 보고, `volatile-*`는 TTL이 있는 key만 대상으로 삼는다.

Cache-only instance라면 approximate LRU나 LFU로 재생성 가능한 key를 제거할 수 있다. TTL 없는 session이나 queue 상태가 같은 instance에 섞여 있으면 policy 선택만으로 안전하게 보호하기 어렵다. 유실 허용 수준이 다른 workload는 instance를 분리하는 편이 명확하다.

## 예시

```text
maxmemory-policy=allkeys-lru
watch: used_memory, evicted_keys, keyspace_hits, miss_latency
alert: evicted_keys 증가 + DB QPS spike
```

`evicted_keys`가 증가하면 key size, TTL 누락과 traffic 변화로 memory가 왜 찼는지 확인한다. Eviction 뒤 cache miss가 database QPS spike로 이어질 수 있으므로 두 지표를 같은 시간축으로 본다.

## 면접 답변 예시

> Redis eviction policy는 `maxmemory`에 도달했을 때 제거할 key 범위와 기준을 정하는 설정입니다. 재생성 가능한 cache라면 allkeys LRU나 LFU를 검토할 수 있지만, session처럼 유실되면 안 되는 상태와 같은 instance에 섞지 않겠습니다. Eviction은 정상 무효화 정책이 아니라 memory pressure의 마지막 안전장치입니다. `evicted_keys`, miss latency와 database QPS를 함께 보고 큰 key와 TTL 누락 원인을 찾습니다.

## 장점

- 재생성 가능한 cache key를 자동 제거해 제한된 memory를 계속 사용할 수 있다.
- Workload에 맞는 LRU·LFU·TTL 기준을 선택할 수 있다.

## 단점

- TTL 없는 상태와 cache를 섞으면 어느 key를 제거해도 다른 요구사항을 해칠 수 있다.
- Eviction으로 miss가 급증하면 origin이 먼저 포화될 수 있다.

## 주의사항 / 실무 팁

- `used_memory`, `evicted_keys`, keyspace miss와 origin QPS를 한 알림 맥락에 둔다.
- 큰 key, TTL 누락과 cache·state workload 혼재 여부를 정기적으로 점검한다.
