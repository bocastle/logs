# Form Validation Constraint API 정리

## 핵심 요약

- native input 제약과 오류 focus를 활용한다.
- client validation만 믿으면 조작된 요청을 막지 못한다.
- 같은 규칙을 서버에서 다시 검증한다.

## 개념 설명

Constraint Validation API는 HTML 폼 컨트롤의 `required`·`type`·`pattern`·`min`·`max` 조건을 `ValidityState`로 표현하고 제출 전 검증하는 브라우저 API다.

`checkValidity()`는 invalid event를 발생시키고 boolean을 반환하며 `reportValidity()`는 브라우저 오류 UI도 보여 준다. custom 규칙은 `setCustomValidity()`로 설정하고 유효해지면 빈 문자열로 지운다.

## 예시

```js
const input = form.elements.email;
input.setCustomValidity(input.value.endsWith("@example.com") ? "" : "회사 이메일을 입력하세요.");
if (!form.reportValidity()) return;
```

회사 도메인 규칙을 custom validity에 넣고 보고 가능한 오류 UI를 표시한다.

## 면접 답변 예시

> Constraint Validation API는 `required`, input `type`, `pattern` 같은 HTML 규칙과 custom validation을 browser의 form 제출 흐름에 연결합니다. `ValidityState`로 실패 이유를 구분하고 `reportValidity()`로 focus와 기본 안내를 활용할 수 있습니다. Custom error는 값이 유효해졌을 때 반드시 빈 문자열로 지워야 하며 제품 언어가 필요하면 별도 메시지와 input의 접근성 관계도 맞추겠습니다. Client 검증은 사용성을 위한 것이므로 같은 규칙을 server에서 다시 검증하고 각 validity 상태를 테스트합니다.

## 장점

- 컨트롤별 invalid 상태를 `ValidityState`로 읽는다.

## 단점

- custom validity를 지우지 않으면 값을 고쳐도 필드가 계속 invalid다.

## 주의사항 / 실무 팁

- input 변경 시 custom validity를 재계산한다.
