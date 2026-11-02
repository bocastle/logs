# Browser Cache Partitioning 정리

## 핵심 요약

- cross-site cache 상태를 이용한 tracking 표면을 줄인다.
- 서로 다른 site 간 공유 CDN cache hit를 기대할 수 없다.
- third-party CDN이 이미 cache됐을 것이라고 성능 budget을 잡지 않는다.

## 개념 설명

Browser Cache Partitioning은 HTTP cache 등 브라우저 저장소의 key에 top-level site 맥락을 추가해 다른 site에서 받은 cross-site resource cache를 공유하지 않게 하는 privacy 보호 모델이다.

동일한 CDN URL이어도 top-level site가 다르면 별도 cache partition에 저장될 수 있다. cache key의 세부 구성과 partition 범위는 브라우저별로 다를 수 있다.

## 예시

```text
(top-level site A, cdn.example/app.js) !=
(top-level site B, cdn.example/app.js)
```

A와 B가 같은 CDN script를 쓰더라도 서로의 cache hit 여부를 관찰하는 tracking 단서로 쓰지 못하게 partition한다.

## 면접 답변 예시

> Browser cache partitioning은 HTTP cache 같은 저장소 key에 top-level site 맥락을 더해 다른 site의 third-party cache 상태를 공유하지 않게 하는 privacy 모델입니다. 같은 CDN URL이라도 다른 top-level site에서 warm cache hit가 보장되지 않으므로 공용 library가 이미 받아져 있을 것이라는 성능 가정을 두면 안 됩니다. Partition 범위는 browser마다 달라 자체 origin bundling, preload와 long-lived cache의 실제 비용을 cold 조건에서 측정하겠습니다. 이 동작을 우회하려 하지 않고 privacy 경계를 product performance budget에 반영합니다.

## 장점

- third-party resource 저장을 top-level 맥락으로 격리한다.

## 단점

- 첫 방문의 third-party resource transfer가 늘 수 있다.

## 주의사항 / 실무 팁

- 브라우저·top-level site·resource origin을 로그 맥락에 포함한다.
