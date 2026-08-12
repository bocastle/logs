# Stale-if-error Cache 정리

## 핵심 요약

- 원본 장애 중에도 읽기 API의 가용성을 높일 수 있다.
- 오래된 데이터가 잘못된 의사결정을 만들 수 있다.
- 데이터별 stale 허용 시간을 문서화한다.

## 개념 설명

Stale-if-error Cache는 원본이 실패할 때 만료된 캐시 값을 제한된 시간 동안 대신 제공해 장애 영향을 줄이는 전략이다.

캐시 값에 fresh TTL과 stale 허용 시간을 나눠 두고, origin 오류나 timeout이면 stale window 안의 값을 반환하면서 재검증을 지연한다.

## 예시

```http
Cache-Control: max-age=60, stale-if-error=300

origin 500 -> serve stale object age=90s
watch: stale_served_count, origin_error_rate
```

stale 응답은 의도적인 품질 저하다. 결제 상태처럼 최신성이 중요한 데이터에는 범위를 매우 좁혀야 한다.

## 면접 답변 예시

> `stale-if-error`는 cached response가 만료됐더라도 origin 오류가 발생하면 정해진 시간 동안 이전 값을 제공하는 정책입니다. 짧은 장애의 가용성을 높일 수 있지만 데이터별로 허용할 stale 시간을 명시해야 합니다. 결제 상태와 개인화 응답에는 적용 범위를 매우 좁게 잡고 cache-control이 사용자 사이에 공유되지 않는지 확인합니다. Origin 오류율과 stale 제공 수를 함께 관찰해 장애가 cache에 가려지지 않게 합니다.

## 장점

- 짧은 origin 장애가 사용자 오류로 바로 드러나는 것을 줄인다.

## 단점

- origin 장애가 캐시에 가려져 늦게 발견될 수 있다.

## 주의사항 / 실무 팁

- stale 응답에는 관측 태그나 header를 남긴다.
