# CSS math functions 정리

## 핵심 요약

- CSS만으로 유연한 레이아웃 계산을 표현할 수 있다.
- 단위 조합 결과가 computed 단계까지 직관적이지 않을 수 있다.
- 계산식에는 중간 custom property 이름을 붙인다.

## 개념 설명

CSS math functions는 `calc()`, `min()`, `max()`, `clamp()` 등으로 길이, 비율, 숫자 값을 런타임에 계산하는 CSS 함수군이다.

`calc()`는 단위가 다른 값을 조합할 때 기본이 되고, `min()`, `max()`, `clamp()`는 계산 결과의 경계를 정해 토큰을 안전한 범위에 묶는다.

## 예시

```css
.panel {
  --gap: clamp(1rem, 2vw, 2rem);
  block-size: calc(100dvh - var(--gap) * 2);
}
```

`calc()`와 `clamp()`를 함께 써 동적 viewport 높이에서 안전한 패널 크기를 계산하는 예다.

## 면접 답변 예시

> CSS math function은 viewport와 container가 바뀔 때 길이·비율을 browser 계산에 맡겨 JavaScript resize logic을 줄이는 방법입니다. `calc()`로 다른 단위를 조합하고 `min()`, `max()`, `clamp()`로 design의 최소·최대 범위를 명시하겠습니다. 복잡한 식은 중간 custom property에 목적을 이름 붙여 computed value를 읽기 쉽게 만듭니다. 새 연산이나 함수는 target browser 지원과 invalid 선언의 fallback을 확인하고 대표 viewport의 실제 값을 test합니다.

## 장점

- JS resize 계산을 줄이고 선언적 토큰을 남긴다.

## 단점

- 복잡한 수식은 디자인 의도를 숨긴다.

## 주의사항 / 실무 팁

- `calc()` 안 곱셈과 나눗셈 지원 범위를 확인한다.
