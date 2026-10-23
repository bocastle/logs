# Database Partition Rebalancing 정리

## 핵심 요약

- storage와 QPS skew를 완화한다.
- 복사 traffic이 network와 disk를 포화시킨다.
- partition movement에 bandwidth throttling을 둔다.

## 개념 설명

partition rebalancing은 node별 데이터 크기나 QPS skew를 줄이기 위해 partition ownership과 데이터를 cluster 안에서 이동하는 작업이다.

source snapshot 복사, 변경분 catch-up, routing cutover, source 정리 순서로 partition movement를 진행하고 throttling으로 foreground 부하를 보호한다.

## 예시

```text
partition p42 source=A target=D
copy snapshot -> stream delta -> verify checksum
route epoch 18 to D -> drain A -> delete old copy
```

routing epoch 없이 source와 target이 동시에 write를 받으면 split ownership과 누락이 생기므로 cutover를 원자적으로 배포해야 한다.

## 면접 답변 예시

> Partition rebalancing은 node별 storage나 QPS skew를 줄이기 위해 partition의 data와 ownership을 다른 node로 옮기는 작업입니다. Snapshot copy, delta catch-up과 checksum 검증 뒤 routing epoch를 원자적으로 바꿔 source와 target이 동시에 owner가 되는 상황을 막겠습니다. Copy traffic은 foreground disk와 network를 포화시키고 이동 중 replica redundancy도 낮아질 수 있습니다. 동시 movement와 bandwidth를 제한하고 cutover 전후 row count, checksum, error와 latency를 확인한 뒤 old copy를 지웁니다.

## 장점

- node 증설 용량을 실제 key range에 분배한다.

## 단점

- cutover 오류는 dual write나 누락을 만든다.

## 주의사항 / 실무 팁

- cutover 전후 row count와 checksum을 비교한다.
