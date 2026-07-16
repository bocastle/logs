# Sanitizer API 도입 전 확인할 것 정리

## 핵심 요약

- HTML Sanitizer API는 브라우저가 HTML을 파싱하면서 허용되지 않거나 XSS에 위험한 요소와 속성을 제거하게 한다.
- 2026년 현재 지원 범위가 제한적이므로 미지원 환경의 검증된 fallback과 같은 보안 결과를 내는지 확인한다.
- 보통은 안전하지 않은 항목을 추가로 제거하는 `setHTML()`을 사용하고 allowlist는 서비스에 필요한 최소 범위로 유지한다.

## 개념 설명

HTML Sanitizer API는 문자열을 DOM에 삽입하거나 문서로 파싱할 때 허용되지 않은 요소와 속성을 제거하는 플랫폼 API다. `Element.setHTML()` 같은 안전 삽입 메서드는 custom 설정에 위험 항목이 들어 있어도 XSS-unsafe 요소를 추가로 제거한다.

도입 전에는 지원 브라우저와 기본 정책, custom `Sanitizer` 설정을 확인한다. 서버에서 이미 정제하더라도 클라이언트가 다른 소스의 HTML을 표시할 수 있다면 책임 경계를 문서화한다. 미지원 환경에서는 DOMPurify 같은 검증된 라이브러리를 사용하되 같은 fixture로 결과 차이를 감시한다.

## 예시

```ts
const sanitizer = new Sanitizer({
  elements: [
    "p",
    "strong",
    { name: "a", attributes: ["href"] },
  ],
  attributes: ["title"],
});
target.setHTML(userHtml, { sanitizer });
```

허용할 요소와 속성을 좁힌 재사용 가능한 `Sanitizer`를 안전한 `setHTML()` 경로에 연결한다. 링크의 scheme과 `target="_blank"`에 필요한 `rel` 같은 의미 검사는 별도 정책이 필요할 수 있다.

## 면접 답변 예시

> HTML Sanitizer API는 HTML을 파싱하면서 허용되지 않거나 XSS에 위험한 요소와 속성을 제거하는 플랫폼 기능입니다. 일반적인 삽입에는 안전한 `setHTML()`을 사용하고 필요한 요소만 허용하는 `Sanitizer`를 재사용하겠습니다. 아직 모든 브라우저에서 사용할 수 없으므로 미지원 환경은 검증된 라이브러리로 처리하고 같은 공격 fixture에서 결과를 비교합니다. 서버 저장 단계와 클라이언트 표시 단계 중 어디에서 어떤 정책을 적용하는지도 명확히 나눠야 합니다.

## 장점

- 브라우저의 HTML 파서와 안전 삽입 메서드를 사용해 직접 문자열 치환보다 견고하게 정제할 수 있다.
- 삽입 지점별 허용 요소와 속성을 재사용 가능한 정책으로 명시할 수 있다.

## 단점

- 지원하지 않는 브라우저의 fallback과 제거 결과가 달라 화면이나 보안 정책이 갈릴 수 있다.
- 허용 목록을 넓게 잡으면 XSS 외에도 phishing에 쓰일 수 있는 링크와 표현이 남을 수 있다.

## 주의사항 / 실무 팁

- URL scheme, `target`, `rel`과 사용자 콘텐츠에 허용할 `data-*` 범위를 함께 검토한다.
- 플랫폼 API와 fallback sanitizer에 같은 공격 fixture를 적용하고 제거 결과 차이를 배포 전에 확인한다.
