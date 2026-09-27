# Accessible Name 계산 정리

## 핵심 요약

- icon-only control의 목적을 보조 기술에 전달한다.
- `aria-label`이 보이는 텍스트와 다르면 voice control 사용자가 호출하기 어렵다.
- native label·button text·`alt`를 우선한다.

## 개념 설명

Accessible Name은 보조 기술이 button·link·input·region의 목적을 알 수 있도록 role별 명명 규칙으로 계산된 문자열이다.

계산 algorithm은 요소의 role과 naming prohibition을 확인한 뒤 `aria-labelledby` 참조 텍스트, `aria-label`, native `label`/`alt`, name from content 등을 정의된 우선순위로 평가한다.

## 예시

```html
<button aria-labelledby="save-label">
  <svg aria-hidden="true">…</svg>
  <span id="save-label">저장</span>
</button>
```

장식 icon은 이름 계산에서 제외하고 참조된 보이는 텍스트 `저장`을 button의 accessible name으로 삼는다.

## 면접 답변 예시

> 보이는 label과 보조 기술 이름을 일치시킨다. name을 허용하지 않는 role에 ARIA를 추가해도 의미가 없다. 다국어·icon loading failure·hidden label 상태를 테스트한다.

## 장점

- testing library와 automation이 사용자가 듣는 이름으로 요소를 찾는다.

## 단점

- 여러 label source를 중복하면 예상보다 긴 이름이 만들어질 수 있다.

## 주의사항 / 실무 팁

- DevTools에서 computed name과 name source를 확인한다.
