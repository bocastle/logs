# CSS counters 정리

## 핵심 요약

- 반복되는 번호 표시를 CSS 구조에 맡길 수 있다.
- CSS로 만든 번호가 복사나 접근성 이름에 기대대로 포함되지 않을 수 있다.
- 의미가 중요한 번호는 실제 텍스트나 list semantics도 확인한다.

## 개념 설명

CSS counters는 CSS가 문서 구조를 따라 번호를 증가시키고 `content`에 표시하게 하는 기능이다.

`counter-reset`, `counter-increment`, `counter()`를 조합해 목차, 단계 번호, 섹션 번호를 DOM 텍스트 추가 없이 만들 수 있다.

## 예시

```css
.steps { counter-reset: step; }
.steps > li::before {
  counter-increment: step;
  content: counter(step) ". ";
}
```

`counter-reset`, `counter-increment`, `counter()`로 단계 번호를 시각적으로 붙이는 예다.

## 면접 답변 예시

> CSS counters는 문서 구조에 따라 번호를 reset하고 증가시켜 목차나 장식용 단계 번호를 자동 표시하는 기능입니다. DOM 순서가 바뀌면 번호도 따라가므로 직접 문자열을 관리하는 중복을 줄일 수 있습니다. 다만 pseudo-element의 generated content는 복사와 보조기술에서 일관되게 의미로 전달된다고 가정하면 안 됩니다. 순서 자체가 중요한 목록은 `ol` 같은 native semantics를 우선하고 counter는 시각 표현에 사용하며 실제 screen reader와 복사 결과를 확인합니다.

## 장점

- 중첩 목차나 절 번호를 자동으로 맞출 수 있다.

## 단점

- DOM 순서와 시각 번호가 다르면 혼란이 생긴다.

## 주의사항 / 실무 팁

- 중첩 counter 이름을 분리한다.
