# CSS color-mix() 정리

## 핵심 요약

- `color-mix()`는 지정한 color space에서 두 색을 비율로 섞어 runtime 파생 색을 만든다.
- 같은 입력도 sRGB와 OKLab에서 결과가 다르므로 design token이 사용할 혼합 공간을 명시한다.
- 계산된 최종 배경·텍스트 조합의 contrast를 실제 theme별로 검증한다.

## 개념 설명

CSS `color-mix()`는 browser가 두 color를 지정한 interpolation space에서 혼합하게 한다. Hover, border와 약한 background를 기준 token에서 파생해 별도 색 token 수를 줄일 수 있다.

`color-mix(in oklab, ...)`처럼 color space와 비율을 함께 적는다. 한쪽 비율을 생략하면 남은 비율을 기준으로 계산되며 합이 100%가 아닐 때의 normalization과 alpha 결과도 단순한 opacity 적용과 다를 수 있다.

## 예시

```css
.badge {
  background: color-mix(in oklab, var(--brand) 18%, transparent);
  border-color: color-mix(in oklab, var(--brand) 55%, Canvas);
}
```

Brand color로 투명 배경과 system `Canvas`에 섞인 border를 만든다. Dark theme에서 `Canvas`와 brand token이 달라질 때도 결과가 읽기 좋은지 확인한다.

## 면접 답변 예시

> `color-mix()`는 지정한 color space에서 두 색을 비율로 섞어 파생 색을 만드는 함수입니다. 같은 색도 sRGB와 OKLab의 결과가 다르므로 design system에서 사용할 혼합 공간을 고정하겠습니다. 이 함수로 만든 배경이 자동으로 접근성 contrast를 만족하는 것은 아니므로 light·dark theme의 computed color를 검증합니다. 미지원 환경에는 앞선 기본 color 선언을 남깁니다.

## 장점

- Theme 전환 시 기준 color에서 배경·border variant를 자동으로 재계산할 수 있다.
- 반복되는 파생 color token 수를 줄일 수 있다.

## 단점

- Color space 선택에 따라 예상한 색조와 명도가 달라질 수 있다.
- Transparent 혼합 결과가 단순 alpha 조정과 다를 수 있다.

## 주의사항 / 실무 팁

- Light·dark와 high contrast theme에서 최종 computed color의 contrast를 검증한다.
- 기본 color를 먼저 선언하고 `@supports` 또는 cascade로 fallback 결과를 확인한다.
