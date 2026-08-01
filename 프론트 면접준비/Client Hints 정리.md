# Client Hints 정리

## 핵심 요약

- Client Hints는 서버가 요청한 client 정보를 브라우저가 `Sec-CH-*` header로 보내는 HTTP 협상 방식이다.
- 첫 요청에는 힌트가 없을 수 있어 기본 응답을 준비하고, 실제 표현을 바꾸는 힌트만 요청한다.
- 응답 variant가 달라진다면 cache key와 개인정보·fingerprinting 비용을 함께 검토한다.

## 개념 설명

Client Hints는 user agent, 화면과 기기 특성 중 서버가 요청한 정보를 브라우저가 `Sec-CH-*` 요청 header로 전달하는 기능이다. 서버는 응답의 `Accept-CH`로 이후 요청에 필요한 힌트를 알린다.

협상이 응답 뒤에 시작되므로 첫 navigation이나 새 origin 요청에는 값이 없을 수 있다. 힌트가 없다고 실패하지 말고 합리적인 기본 표현을 보낸다. Cross-origin frame에 전달할 때는 Permissions Policy를 검토하고, 응답이 달라지는 header는 CDN cache key와 `Vary` 정책에 맞춘다.

## 예시

```http
Accept-CH: Sec-CH-DPR, Sec-CH-Viewport-Width
Permissions-Policy: ch-dpr=(self), ch-viewport-width=(self)
```

자체 origin에 DPR과 viewport width를 요청하는 예다. 모든 페이지가 두 값을 실제로 사용하지 않는다면 범위를 더 좁히고 지원하지 않는 browser의 fallback을 확인한다.

## 면접 답변 예시

> Client Hints는 서버가 요청한 client 정보를 브라우저가 `Sec-CH-*` header로 보내는 협상 방식입니다. `Accept-CH`는 보통 이후 요청에 영향을 주므로 첫 요청에는 힌트가 없을 수 있습니다. 실제 응답 선택에 필요한 값만 요청하고 variant가 달라지면 CDN cache key에도 반영하겠습니다. 힌트가 없거나 지원되지 않는 환경의 기본 응답, Permissions Policy와 fingerprinting 영향을 함께 검토합니다.

## 장점

- HTML parsing 뒤 JavaScript로 탐지하기 전에 서버가 variant를 선택할 수 있다.
- Hint 사용 계약을 HTTP header와 cache 정책에 명시할 수 있다.

## 단점

- Hint 조합으로 응답 variant가 많아지면 CDN cache hit ratio가 낮아진다.
- 과도한 client 정보 수집은 fingerprinting 표면과 요청 크기를 늘린다.

## 주의사항 / 실무 팁

- Hint가 없을 때의 fallback을 먼저 만들고 실제 사용하는 조합만 cache key에 반영한다.
- Browser와 CDN에서 request header, 선택 variant와 hit ratio를 함께 확인한다.
