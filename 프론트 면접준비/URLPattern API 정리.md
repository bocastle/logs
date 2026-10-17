# URLPattern API 정리

## 핵심 요약

- 라우팅 패턴을 URL 구조와 가깝게 표현할 수 있다.
- 브라우저 지원 범위가 제한적일 수 있다.
- 지원 여부를 확인하고 기존 라우터 matcher fallback을 둔다.

## 개념 설명

URLPattern API는 URL의 protocol, host, pathname, search를 선언형 패턴으로 매칭하는 브라우저 플랫폼 API다.

`new URLPattern`으로 경로 변수와 패턴을 만들고 `exec` 결과에서 그룹 값을 읽어 라우팅이나 요청 필터링에 사용할 수 있다.

## 예시

```ts
const pattern = new URLPattern({ pathname: "/posts/:id" });
const match = pattern.exec("https://example.com/posts/42");
const postId = match?.pathname.groups.id;
```

문자열 split 대신 URL 구조를 기준으로 `:id` 그룹을 얻는다. 지원하지 않는 환경에는 라우터의 자체 matcher가 필요하다.

## 면접 답변 예시

> URLPattern API는 protocol, hostname과 pathname을 string split 대신 URL 구조에 맞는 pattern으로 매칭하는 기능입니다. Route나 Service Worker request 분기에 사용할 수 있지만 지원하지 않는 환경에는 기존 router matcher fallback이 필요합니다. Relative input은 base URL을 명시해야 하는 경우가 있고 encoded path, optional segment와 trailing slash 정책도 먼저 정해야 합니다. 절대·상대 URL, 다른 origin과 decode 경계를 test해 pattern이 의도보다 넓게 match하지 않는지 확인합니다.

## 장점

- pathname 그룹 추출을 표준 API로 처리한다.

## 단점

- 정규식과 패턴 문법을 섞으면 읽기 어려워진다.

## 주의사항 / 실무 팁

- 경로 변수 이름과 trailing slash 규칙을 문서화한다.
