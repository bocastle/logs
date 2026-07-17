# Client Hints로 이미지 최적화 요구사항 잡는 법 정리

## 핵심 요약

- 이미지 Client Hints는 서버가 DPR과 요청 폭을 참고해 적절한 이미지 variant를 선택하게 한다.
- 힌트를 사용해 응답이 달라진다면 CDN cache key 또는 `Vary`에 같은 요청 header 축을 반영한다.
- 지원하지 않는 요청을 위한 기본 이미지와 `srcset`·`sizes`만으로 해결할 수 있는지 먼저 검토한다.

## 개념 설명

Client Hints 이미지 최적화는 브라우저가 DPR, viewport와 리소스의 예상 폭을 request header로 보내고 서버나 CDN이 적절한 크기의 이미지를 고르는 방식이다. Origin은 `Accept-CH`로 필요한 힌트를 요청한다.

힌트를 많이 요청할수록 cache variant 조합과 운영 복잡도가 커진다. 실제 응답 선택에 사용하는 header만 요청하고 cache key에 반영한다. 첫 요청이나 미지원 환경에는 힌트가 없을 수 있으므로 합리적인 기본 이미지를 제공해야 한다.

## 예시

```http
Accept-CH: Sec-CH-DPR, Sec-CH-Viewport-Width, Sec-CH-Width
Vary: Sec-CH-DPR, Sec-CH-Viewport-Width, Sec-CH-Width
```

`Accept-CH`와 `Vary`에 실제 `Sec-CH-*` 요청 header 이름을 일치시킨 예다. CDN이 `Vary`를 그대로 cache key에 반영하는지는 제품 설정으로 따로 확인하고, 폭 값을 몇 개 구간으로 정규화해 variant 수를 제한한다.

## 면접 답변 예시

> 이미지 Client Hints는 브라우저의 DPR과 요청 폭을 서버에 전달해 적절한 variant를 고르게 하는 방식입니다. Origin은 필요한 `Sec-CH-*` header만 `Accept-CH`로 요청하고, 응답이 달라지는 축은 CDN cache key나 `Vary`에 반영해야 합니다. 힌트가 없는 첫 요청과 미지원 브라우저에는 기본 이미지를 제공하고 폭을 몇 개 구간으로 정규화해 cache fragmentation을 줄입니다. 단순한 화면이라면 `srcset`과 `sizes`만으로 충분한지도 먼저 비교하겠습니다.

## 장점

- 기기와 레이아웃에 필요한 해상도에 가까운 이미지를 보내 byte 낭비를 줄일 수 있다.
- Variant 선택을 서버와 CDN의 공통 정책으로 운영할 수 있다.

## 단점

- 힌트와 원본 값 조합이 많으면 cache fragmentation과 변환 비용이 커진다.
- 힌트가 없는 요청의 기본값을 잘못 고르면 첫 화면의 화질이나 전송량이 나빠질 수 있다.

## 주의사항 / 실무 팁

- 요청 힌트, 정규화한 폭 구간과 선택 variant를 로그에 남겨 hit ratio와 전송량을 비교한다.
- CDN이 `Vary`를 실제 cache key에 반영하는지 배포 환경에서 확인한다.
