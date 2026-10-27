# Consistent Hashing Virtual Node 튜닝 정리

## 핵심 요약

- node 추가·삭제 시 일부 key만 이동한다.
- virtual node가 너무 많으면 metadata와 repair overhead가 커진다.
- key count와 bytes·QPS 분포를 각각 측정한다.

## 개념 설명

consistent hashing virtual node는 한 physical node를 hash ring의 여러 token으로 표현해 key와 replica 분포를 고르게 만드는 단위다.

virtual node 수를 늘리면 평균 분포는 균등해지지만 routing metadata와 partition movement 조각 수가 늘어 node weight와 failure domain을 함께 반영해야 한다.

## 예시

```text
node A weight=2 -> 256 virtual nodes
node B weight=1 -> 128 virtual nodes
replica placement skips same rack failure domain
```

key 자체가 hot하면 virtual node를 늘려도 단일 key 부하는 분산되지 않으므로 key splitting이나 request coalescing이 따로 필요하다.

## 면접 답변 예시

> Consistent hashing의 virtual node는 physical node를 ring의 여러 token으로 나눠 node 추가·삭제 때 일부 key만 이동시키면서 분포를 고르게 하는 방법입니다. Token 수를 늘리면 평균 균등성은 좋아지지만 metadata, repair와 movement 조각 수가 커집니다. Capacity가 다른 node는 weight를 반영하고 replica placement는 같은 rack·zone에 몰리지 않게 하겠습니다. Key count뿐 아니라 bytes와 QPS skew를 측정하며 단일 hot key는 virtual node가 아니라 key split이나 request coalescing으로 해결합니다.

## 장점

- physical node 용량 비율로 token 수를 조정한다.

## 단점

- 잘못된 token 배치는 rack 장애에 replica를 함께 잃는다.

## 주의사항 / 실무 팁

- node weight 변경 전 이동량을 시뮬레이션한다.
