# CSS custom property fallback 정리

## 핵심 요약

- 테마 토큰 누락이 곧바로 깨진 UI로 이어지는 일을 줄인다.
- fallback이 제품 대비 기준을 만족하지 않을 수 있다.
- 핵심 색과 간격 토큰에는 의미 있는 fallback을 둔다.

## 개념 설명

CSS custom property fallback은 `var()` 안의 두 번째 인자로 참조한 토큰이 정의되지 않았거나 CSS-wide keyword로 유효한 custom property 값을 만들지 못할 때 대체 값을 지정하는 방식이다.

`var(--button-bg, Canvas)`처럼 fallback을 넣으면 테마 토큰 누락 시에도 기본 스타일이 남는다. fallback 자체가 invalid이면 전체 선언이 무효가 된다.

## 예시

```css
.button {
  color: var(--button-fg, CanvasText);
  background: var(--button-bg, Canvas);
}
```

`var()` fallback으로 `--button-*` 토큰이 없을 때도 읽을 수 있는 색을 유지하는 예다.

## 면접 답변 예시

> `var(--token, fallback)`은 token이 없을 때 component 기본값을 제공해 점진적 theme migration을 돕습니다. 다만 token이 존재하지만 `--gap: red`처럼 실제 `padding` 문법에 맞지 않으면 fallback이 선택되는 것이 아니라 사용한 선언이 computed-value 단계에서 무효가 될 수 있습니다. 핵심 token은 typed `@property`나 test로 값 범위를 검증하고 fallback 자체도 contrast와 layout 기준을 만족하게 하겠습니다. Token 이름 변경이 끝난 뒤 임시 fallback이 오류를 오래 숨기지 않는지도 정리합니다.

## 장점

- 컴포넌트 기본값을 CSS 선언 안에 남길 수 있다.

## 단점

- 중첩 fallback이 길어지면 선언이 읽기 어렵다.

## 주의사항 / 실무 팁

- fallback 값도 contrast와 layout 기준으로 검수한다.
