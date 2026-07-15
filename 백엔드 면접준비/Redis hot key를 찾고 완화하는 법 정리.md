# Redis hot key를 찾고 완화하는 법 정리

## 핵심 요약

- Hot key는 전체 메모리가 아니라 특정 key의 요청이 한 shard의 CPU와 네트워크에 몰리는 문제다.
- 평균 Redis 지연만 보지 말고 shard별 CPU·network와 key별 QPS를 같은 시간대로 맞춰 원인을 찾는다.
- 읽기 복제, 짧은 local cache와 key 분할은 데이터 일관성과 무효화 비용을 비교해 선택한다.

## 개념 설명

Redis cluster에서는 key 하나가 하나의 hash slot과 shard로 간다. 특정 상품이나 설정 key에 요청이 몰리면 전체 cluster에 여유가 있어도 그 shard의 CPU와 network가 먼저 포화될 수 있다.

서버 command 통계와 sampling만으로는 짧은 burst를 놓칠 수 있어 클라이언트에서 key를 안전하게 정규화한 QPS 지표도 수집한다. 읽기 중심이면 replica와 짧은 local cache를 검토하고, key 분할이 필요하면 어떤 복사본을 읽고 모든 복사본을 어떻게 무효화할지 먼저 정한다.

## 예시

```text
hot key: product:main-banner
qps=45k, shard cpu=92%
mitigation: product:main-banner:{0..9} + short local cache
watch: top_key_qps, shard_cpu, hit_latency
```

예시의 열 개 key는 같은 값을 복제해 읽기 부하를 분산하는 전략이다. Cluster hash tag를 잘못 사용하면 중괄호 안 값 때문에 다시 같은 slot에 모일 수 있으므로 실제 slot 분포를 확인해야 한다.

## 면접 답변 예시

> Redis hot key는 특정 key의 요청이 한 shard에 집중돼 CPU나 network가 병목이 되는 상태입니다. 평균 지연보다 shard별 사용량과 상위 key QPS를 같은 시간대에 비교해 찾겠습니다. 읽기 중심이라면 짧은 local cache나 replica를 먼저 검토하고, key를 여러 복사본으로 나눌 때는 slot 분포와 무효화 경로를 함께 설계합니다. 적용 뒤에는 stale 비율과 shard 간 부하가 실제로 평준화됐는지 확인합니다.

## 장점

- Cluster 전체 평균에 가려진 shard별 부하 불균형을 원인 key까지 좁힐 수 있다.
- 읽기 중심 key는 짧은 local cache만으로도 Redis 요청을 크게 줄일 수 있다.

## 단점

- Local cache와 replica 읽기는 stale 응답 가능성을 높인다.
- Key 복제와 sharding은 읽기 선택, 쓰기 fan-out과 무효화 경로를 복잡하게 만든다.

## 주의사항 / 실무 팁

- 쓰기 빈도가 낮고 읽기가 높은 key부터 짧은 local cache를 검토한다.
- Key 복사본이 실제로 다른 hash slot에 배치되는지와 무효화 누락을 부하 테스트로 확인한다.
