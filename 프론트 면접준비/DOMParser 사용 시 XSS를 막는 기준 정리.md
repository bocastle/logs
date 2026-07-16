# DOMParser 사용 시 XSS를 막는 기준 정리

## 핵심 요약

- `DOMParser`는 문자열을 DOM으로 파싱할 뿐 공격용 태그와 속성을 안전하게 제거하지 않는다.
- 신뢰할 수 없는 HTML은 검증된 sanitizer의 좁은 허용 정책을 거친 뒤 실제 문서에 삽입한다.
- Trusted Types와 CSP는 위험한 sink로 문자열이 직접 들어가는 경로를 줄이지만 sanitizer 정책 자체를 대신하지는 않는다.

## 개념 설명

`DOMParser.parseFromString(..., "text/html")`은 입력을 별도 문서로 파싱한다. 파싱 중 script가 바로 실행되지 않는다고 해서 결과가 안전한 것은 아니다. 이벤트 handler, 위험한 URL과 DOM clobbering용 이름은 남아 있을 수 있고 그 노드를 실제 문서에 옮기는 순간 문제가 된다.

따라서 신뢰할 수 없는 HTML은 DOMPurify처럼 검증된 sanitizer와 서비스별 allowlist를 거친다. Trusted Types를 적용하면 승인된 정책에서 만든 값만 관련 sink에 전달하도록 강제할 수 있다. 정책을 여러 개 만들거나 공격자 입력으로 정책을 고르게 하면 통제 지점이 다시 흐려진다.

## 예시

```ts
const policy = trustedTypes.createPolicy("comment-html", {
  createHTML: (value) => DOMPurify.sanitize(value),
});
const safeHTML = policy.createHTML(untrustedHTML);
const doc = new DOMParser().parseFromString(safeHTML, "text/html");
preview.replaceChildren(...doc.body.childNodes);
```

정제 함수를 하나의 Trusted Types 정책 안에 두고 그 결과만 파싱하는 흐름이다. 링크의 protocol, `id`·`name`, SVG와 MathML 허용 여부처럼 서비스 정책에 따라 달라지는 입력은 회귀 fixture로 검증한다.

## 면접 답변 예시

> `DOMParser`는 HTML을 별도 문서로 파싱할 뿐 XSS를 막는 sanitizer가 아닙니다. 신뢰할 수 없는 문자열은 검증된 sanitizer의 좁은 allowlist를 거친 뒤 DOM으로 옮기겠습니다. Trusted Types와 CSP를 적용하면 승인되지 않은 문자열이 위험한 sink로 들어가는 경로를 줄일 수 있지만 정제 정책 자체는 여전히 필요합니다. 이벤트 속성, `javascript:` URL, DOM clobbering과 SVG 입력을 회귀 테스트로 확인합니다.

## 장점

- 정제와 파싱 책임을 분리해 보안 검토 지점을 명확하게 만들 수 있다.
- Trusted Types로 일반 문자열이 위험한 DOM sink에 직접 들어가는 경로를 줄일 수 있다.

## 단점

- 자체 정규식 sanitizer는 브라우저 HTML 파서의 오류 복구와 중첩 규칙을 놓치기 쉽다.
- 허용 정책이 넓거나 여러 정책이 제각각이면 정제 단계를 거쳐도 우회 경로가 남을 수 있다.

## 주의사항 / 실무 팁

- `href`와 `src`의 protocol, 이벤트 속성, SVG, `id`·`name` 충돌을 포함한 회귀 fixture를 둔다.
- CSP의 Trusted Types 위반 report를 먼저 관찰한 뒤 enforcement를 단계적으로 적용한다.
