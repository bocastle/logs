# CSS cascade layer를 디자인 시스템에 적용하는 법 정리

## 핵심 요약

- layer 순서로 스타일 책임을 문서화한다.
- layer 순서 선언이 누락되면 import 순서에 따라 우선순위가 바뀐다.
- entry stylesheet에 전체 layer 순서를 한 번 선언한다.

## 개념 설명

디자인 시스템의 cascade layer 설계는 reset, tokens, base, components, utilities, overrides의 우선순위를 specificity 경쟁 대신 명시적 순서로 관리하는 방식이다.

`@layer reset, base, components, utilities;`처럼 순서를 먼저 고정하면 normal declaration은 나중 layer가 앞선다. unlayered normal style은 layered normal style보다 앞서고 important layer 순서는 반대다.

## 예시

```css
@layer reset, base, components, utilities;
@layer components { :where(.button) { padding: .75rem 1rem; } }
@layer utilities { .p-0 { padding: 0; } }
```

component 기본값은 낮은 specificity로 두고 utilities layer가 의도적으로 덮을 수 있게 한다.

## 면접 답변 예시

> 서드파티 CSS를 낮은 layer에 격리하기 쉬워진다. important 순서 반전을 모르면 긴급 override가 어긋난다. layer 도입 전후 computed style 회귀를 상태별로 비교한다.

## 장점

- specificity 올리기 경쟁을 줄인다.

## 단점

- unlayered CSS가 예상보다 강해 layered component를 덮을 수 있다.

## 주의사항 / 실무 팁

- 각 layer의 소유 책임과 import 규칙을 정한다.
