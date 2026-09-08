# DOMParser와 안전한 HTML 파싱 정리

## 핵심 요약

- 문자열 조작보다 HTML 구조를 기준으로 변환할 수 있다.
- 선택자에 없는 자식만 골라 옮기면 원래 문서 계층이 깨질 수 있다.
- 콘텐츠 종류에 맞춰 text/html과 XML MIME type을 명시한다.

## 개념 설명

DOMParser의 HTML 파싱은 문자열을 별도 `Document`로 구조화해 선택, 검사, 변환하는 단계이며 파싱 결과를 현재 문서에 넣는 단계와 분리해야 한다.

`parseFromString(value, 'text/html')`로 만든 문서에서 허용된 요소만 골라 현재 문서의 `adoptNode`나 복제 과정으로 옮기면 소유 문서와 삽입 위치를 명시할 수 있다.

## 예시

```ts
const parsed = new DOMParser().parseFromString(sanitizedHTML, "text/html");
const fragment = document.createDocumentFragment();
for (const node of parsed.body.childNodes) {
  fragment.append(document.importNode(node, true));
}
preview.replaceChildren(fragment);
```

이미 정책에 따라 정제된 HTML을 별도 문서로 파싱한 뒤 최상위 node부터 현재 document로 복제해 중첩 구조를 보존한다. 허용 목록을 직접 구현한다면 tree를 재귀 순회하며 element와 attribute를 함께 검증해야 한다.

## 면접 답변 예시

> `DOMParser`는 HTML string을 별도 `Document`로 구조화할 뿐 XSS를 막는 sanitizer는 아닙니다. Parsing 결과를 현재 DOM에 넣기 전에 검증된 sanitizer 정책으로 element, attribute와 URL protocol을 정제해야 합니다. 선택된 하위 element를 각각 복제하면 중첩 node가 중복되거나 계층이 깨질 수 있어 최상위 node부터 tree 구조를 보존하겠습니다. XML과 HTML mode의 오류 처리도 다르므로 MIME type과 parser error 사례를 명시적으로 테스트합니다.

## 장점

- 파싱 문서와 표시 문서의 경계를 코드에 드러낼 수 있다.

## 단점

- XML 모드와 HTML 모드를 혼동하면 오류 복구와 대소문자 처리가 달라진다.

## 주의사항 / 실무 팁

- parsererror 처리와 빈 body 처리 사례를 테스트한다.
