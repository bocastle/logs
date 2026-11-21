# Database Shard Key 선택 정리

## 핵심 요약

- query routing을 단일 shard로 제한한다.
- 낮은 cardinality는 큰 hotspot을 만든다.
- 실제 query log로 routing locality를 계산한다.

## 개념 설명

shard key는 row를 어느 shard에 저장하고 query를 어디로 routing할지 결정하는 partitioning key다.

높은 cardinality와 균등한 write 분포, 주요 query locality를 함께 만족해야 하며 key 없는 조건은 모든 shard로 scatter-gather된다.

## 예시

```text
candidate tenant_id: locality 좋음, 큰 tenant hotspot 위험
candidate hash(user_id): 균등, tenant range query scatter-gather
compound tenant_id+bucket: 절충
```

현재 row count만 균등해도 특정 tenant의 미래 성장이나 단조 증가 key가 한 shard write를 집중시킬 수 있다.

## 면접 답변 예시

> Shard key는 row의 저장 위치와 query routing을 결정하므로 cardinality만 높다고 좋은 key는 아닙니다. 실제 query log에서 단일 shard로 끝나는 비율을 계산하고 tenant별 bytes·write QPS, 성장률과 시간대 hotspot을 함께 보겠습니다. `tenant_id`는 locality가 좋지만 큰 tenant가 한 shard를 압박할 수 있고 hash key는 균등한 대신 tenant range query가 scatter-gather가 될 수 있습니다. 큰 tenant 분할, compound key와 rebalancing을 미리 시뮬레이션하고 application이 물리 shard를 직접 알지 않도록 logical routing layer를 둡니다.

## 장점

- storage와 write QPS를 수평 분산한다.

## 단점

- key 변경은 대규모 data movement를 요구한다.

## 주의사항 / 실무 팁

- 상위 key의 bytes와 QPS skew를 시뮬레이션한다.
