# CDN cache key를 설계하는 기준 정리

## 핵심 요약

- CDN cache key는 요청 중 어떤 차이를 같은 캐시 객체로 취급할지 정하는 규칙이다.
- 먼저 공개 응답과 개인화 응답을 분리하고, 공개 캐시가 가능한 응답에만 key 최적화를 적용한다.
- Query와 header를 너무 많이 포함하면 적중률이 떨어지고, 필요한 축을 빼면 잘못된 콘텐츠가 섞인다.

## 개념 설명

CDN cache key는 method와 host, path, query, 일부 요청 header 중 무엇을 조합해 하나의 캐시 객체를 찾을지 정한다. 같은 표현을 반환하는 요청은 같은 key로 모으고, 실제 표현이 다른 요청은 분리해야 한다.

`utm_source` 같은 추적 query는 응답을 바꾸지 않는다면 제거할 수 있다. 반대로 언어와 압축 방식이 표현을 바꾼다면 해당 header를 key 또는 `Vary` 정책에 반영한다. 사용자 cookie나 authorization에 따라 달라지는 응답은 key에 모든 값을 넣기보다 private·bypass 정책부터 검토한다.

## 예시

```text
cache key = GET + /assets/app.js + v=42
ignore query: utm_source, fbclid
vary headers: Accept-Encoding only
watch: hit_ratio, origin_qps, key_count
```

설정 문서만 보고 끝내지 않고 CDN debug header와 실제 요청으로 계산된 key를 확인한다. Query 정렬, 대소문자, 기본 포트처럼 CDN별 정규화 규칙이 다르면 로컬 테스트와 운영 결과가 달라질 수 있다.

## 면접 답변 예시

> CDN cache key는 method, path, query와 일부 header 중 어떤 요청을 같은 객체로 볼지 정하는 규칙입니다. 먼저 개인화 응답을 공개 캐시에서 제외하고, 실제 콘텐츠를 바꾸는 축만 key에 남기겠습니다. 추적 query는 제거할 수 있지만 언어나 압축처럼 표현이 달라지는 값은 분리해야 합니다. 배포 후에는 실제 CDN key, hit ratio, 객체 수와 origin QPS를 함께 확인합니다.

## 장점

- 응답을 바꾸지 않는 query 차이 때문에 캐시가 불필요하게 쪼개지는 일을 막는다.
- 공개 응답의 적중률을 높여 origin 요청과 지연을 줄일 수 있다.

## 단점

- Header와 query 조합이 많으면 캐시 객체 수가 폭발하고 적중률이 낮아진다.
- 개인화 축을 누락하면 다른 사용자의 응답이 공유되는 심각한 정보 노출이 생길 수 있다.

## 주의사항 / 실무 팁

- 배포 전후 CDN debug header로 실제 key 구성과 hit·miss 상태를 확인한다.
- hit ratio만 보지 말고 key 수, eviction, origin QPS와 개인화 응답의 cache-control도 함께 검사한다.
