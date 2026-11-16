# Browser Autofill과 form UX 정리

## 핵심 요약

- 반복 입력 비용을 줄인다.
- autofill 후 클라이언트 상태가 동기화되지 않을 수 있다.
- 마우스 입력 없이 autofill만 수행해 상태를 테스트한다.

## 개념 설명

Browser Autofill UX는 브라우저가 저장한 주소·연락처·인증 정보를 폼에 채운 뒤 검증과 편집이 자연스럽게 이어지도록 만드는 설계다.

브라우저는 `autocomplete`, `name`, `id`, `label`, 폼 구조를 함께 보고 autofill 후보를 정한다. `inputmode`는 autofill이 아니라 모바일 키보드 힌트이므로 두 속성의 책임을 구분한다.

## 예시

```css
.field:has(input:not(:placeholder-shown)) label {
  translate: 0 -1.25rem;
}
```

autofill로 값이 들어와도 label이 값을 가리지 않도록 필드 상태를 CSS와 실제 value로 반영한다.

## 면접 답변 예시

> Browser autofill은 사용자가 반복 입력을 줄일 수 있게 브라우저가 저장한 값을 form에 채우는 기능입니다. `label`, `name`과 목적에 맞는 `autocomplete` token을 제공하고 `inputmode`는 키보드 힌트일 뿐 autofill 설정이 아니라는 점을 구분하겠습니다. 값이 키 입력 없이 채워져도 floating label, validation과 submit 상태가 실제 DOM value를 기준으로 맞아야 합니다. Chrome·Safari와 password manager에서 마우스 입력 없이 자동완성한 뒤 수정, 오류 표시와 제출까지 테스트합니다.

## 장점

- 주소와 결제 폼의 완료율을 높인다.

## 단점

- 브라우저 전용 배경색이 대비를 깨뜨릴 수 있다.

## 주의사항 / 실무 팁

- label이 autofill value를 가리지 않는지 확인한다.
