# HTTP Vary 헤더를 캐시 전략에 반영하는 법 정리

## 핵심 요약

- `Vary`는 캐시가 어떤 요청 헤더 차이를 별도 응답으로 저장해야 하는지 알려 주는 응답 헤더다.
- `Accept-Encoding`, `Accept-Language`처럼 실제 표현을 바꾸는 헤더만 넣어야 캐시 오염을 막으면서 적중률을 지킬 수 있다.
- 사용자별 값이나 값 종류가 지나치게 많은 헤더를 넣으면 캐시 키가 폭발하므로 `Cache-Control`과 함께 설계해야 한다.

## 개념 설명

공유 캐시는 URL이 같아도 `Vary`에 적힌 요청 헤더 값이 다르면 서로 다른 variant로 저장한다. 원본이 헤더에 따라 다른 콘텐츠를 반환하면서 `Vary`를 빠뜨리면 다른 사용자의 표현이 섞일 수 있다.

반대로 `User-Agent`나 추적 ID처럼 값의 종류가 많은 헤더를 넣으면 hit ratio가 급격히 떨어진다. 가능하면 소수의 정규화된 헤더나 URL 차원으로 변형 축을 제한한다.

## 예시

```http
HTTP/1.1 200 OK
Cache-Control: public, max-age=60, stale-while-revalidate=30
Vary: Accept-Encoding, Accept-Language
Content-Language: ko
```

압축 방식과 언어가 실제 응답을 바꾸기 때문에 두 헤더를 variant 키로 사용한다. 인증 쿠키에 따라 달라지는 응답이라면 공유 캐시 허용 여부부터 다시 봐야 한다.

## 면접 답변 예시

> `Vary`는 같은 URL의 응답을 저장할 때 어떤 요청 헤더 차이를 별도 variant로 취급할지 캐시에 알려 주는 응답 헤더입니다. 원본이 언어나 압축 방식에 따라 다른 표현을 보내면 해당 요청 헤더를 포함해야 캐시 오염을 막을 수 있습니다. 반대로 값의 종류가 많은 헤더를 추가하면 variant가 폭증해 적중률이 떨어집니다. 실제 CDN의 cache key 정책과 `Cache-Control`을 함께 확인하고 variant 수, hit ratio, origin QPS를 관찰하겠습니다.

## 장점

- 같은 URL의 언어와 압축 형식을 안전하게 분리할 수 있다.
- 원본이 선택한 표현 기준을 CDN과 브라우저 캐시에 명시해 잘못된 variant가 섞이는 일을 막는다.

## 단점

- 헤더 조합이 많으면 캐시 객체 수와 원본 요청이 빠르게 늘어난다.
- CDN마다 `Vary`와 cache key를 지원하거나 정규화하는 방식이 다를 수 있다.
- 개인화 응답을 잘못 공개 캐시에 두면 사용자 정보가 노출될 수 있다.

## 주의사항 / 실무 팁

- 배포 전에 실제 CDN cache key와 `Vary` 처리 방식을 요청·응답으로 확인한다.
- Variant별 hit ratio와 origin QPS를 함께 보고, `Vary: *`는 일반적인 공유 캐시 재사용을 막는다는 점을 확인한다.
