# Local Cache와 Distributed Cache 정리

## 핵심 요약

- Local cache는 network hop 없이 빠르지만 instance마다 값과 용량이 따로 존재한다.
- Distributed cache는 여러 instance가 같은 값을 공유하지만 network·Redis 장애와 hot key에 의존한다.
- 둘을 계층으로 함께 쓸 때는 stale 허용 시간과 version·무효화 순서를 먼저 정한다.

## 개념 설명

Local cache는 application process memory에 값을 저장해 가장 짧은 latency로 읽는다. 대신 instance마다 cache가 달라 한 node에서 지운 값이 다른 node에는 남을 수 있다. Distributed cache는 Redis 같은 외부 저장소를 공유해 값 재사용 범위가 넓다.

Distributed cache도 database와 자동으로 일관된 것은 아니며 network latency, timeout과 hot key 문제가 생긴다. Local과 distributed를 함께 두면 Redis 부하는 줄지만 database, Redis, 각 process에 세 개의 version이 존재한다. 데이터별 stale 허용 시간과 장애 시 bypass 정책이 선택 기준이다.

## 예시

```text
local cache: product:42 ttl=3s per instance
distributed cache: Redis product:42 ttl=60s
invalidation -> publish version=17
```

예시는 local TTL을 짧게 두고 Redis의 version invalidation을 전달하는 계층형 구조다. Pub/sub event를 놓쳐도 TTL로 회복되게 하고 event version이 이전 값보다 새로울 때만 제거한다.

## 면접 답변 예시

> Local cache는 process 안에서 매우 빠르지만 instance마다 값이 달라 stale 위험이 있고, distributed cache는 값을 공유하는 대신 network와 cache server 장애에 의존합니다. Hot path에는 짧은 local cache를 Redis 앞에 둘 수 있지만 무효화 event를 놓치는 경우까지 고려해 TTL과 version을 함께 사용하겠습니다. 데이터마다 허용할 stale 시간을 먼저 정하고 장애 때 cache를 bypass할지 제한할지 결정합니다. Hit ratio와 latency는 local, distributed와 origin 단계로 나눠 봅니다.

## 장점

- Local cache는 network hop 없이 hot path latency와 Redis hot key 부하를 줄인다.
- Distributed cache는 여러 instance가 같은 값을 재사용할 수 있다.

## 단점

- Local cache는 instance별 stale 상태와 memory 중복을 만든다.
- Distributed cache 장애와 timeout은 모든 호출 instance의 latency를 함께 키울 수 있다.

## 주의사항 / 실무 팁

- Local cache TTL을 짧게 두고 version이 있는 invalidation event를 검토한다.
- Cache별 hit·miss, stale 사례, Redis timeout과 origin QPS를 계층별로 관찰한다.
