# Database Hotspot Write 완화 정리

## 핵심 요약

- row lock과 index latch 경합을 분산한다.
- bucket 합산은 read 비용과 일시 불일치를 늘린다.
- lock wait와 key별 QPS로 hotspot을 먼저 확인한다.

## 개념 설명

hotspot write는 많은 transaction이 같은 row, index page, shard key에 쓰기를 집중해 lock과 log serialization이 생기는 병목이다.

counter를 bucket별로 분산하거나 key salting과 time partitioning으로 write 위치를 나눈 뒤 read에서 합산하면 단일 hot key 경합을 줄인다.

## 예시

```text
views:video42 -> views:video42:{0..63} key salting
writer chooses hash(request_id)%64
reader sums 64 buckets or periodic rollup
```

sharding만 적용해도 동일 shard key의 단일 row는 여전히 한 partition에 모이므로 write aggregation 단위를 실제로 분할해야 한다.

## 면접 답변 예시

> Database write hotspot은 많은 transaction이 같은 row, index page나 shard에 몰려 lock과 log serialization이 생기는 병목입니다. 먼저 key별 QPS, lock wait와 index page 분포로 실제 hotspot을 확인하고 단순히 DB instance만 키우기 전에 경합 대상을 찾겠습니다. 합산 가능한 counter는 bucket·salting이나 append 후 batch rollup으로 분산할 수 있지만 unique inventory처럼 즉시 지켜야 하는 invariant는 같은 방식으로 쪼갤 수 없습니다. Bucket 수는 peak concurrency로 정하고 read 합산 비용, 일시 불일치와 rollup 중복을 부하 테스트하며 최종 정확성 기준을 별도로 둡니다.

## 장점

- 단일 shard의 write throughput 상한을 늘린다.

## 단점

- salting key 수가 과하면 metadata와 cleanup이 커진다.

## 주의사항 / 실무 팁

- bucket 수를 실제 peak concurrency로 정한다.
