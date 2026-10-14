# CSS prefers-contrast 정리

## 핵심 요약

- 사용자 대비 선호에 테마를 적응시킨다.
- query 지원이 없을 수 있으므로 기본 대비를 대체하지 못한다.
- 기본 color token이 WCAG 대비를 먼저 만족하게 한다.

## 개념 설명

CSS `prefers-contrast` media feature는 사용자가 사용자 인터페이스에서 더 높거나 낮은 대비를 선호하는지 CSS에 알린다.

`more`, `less`, `custom`, `no-preference`에 따라 경계선·텍스트·비활성 상태 token을 조정한다. 기본 디자인 자체가 대비 기준을 충족해야 하며 query는 부가 적응이다.

## 예시

```css
@media (prefers-contrast: more) {
  :root { --border-subtle: currentColor; }
  .muted { opacity: 1; }
}
```

더 높은 contrast를 선호하면 약한 border를 현재 text color로 바꾸고 opacity로 낮춘 텍스트를 복구한다.

## 면접 답변 예시

> `prefers-contrast`는 사용자가 UI의 대비를 더 높거나 낮게 선호하는지 알려 주는 media feature입니다. 기본 color token이 먼저 필요한 대비를 충족하게 하고 `more`에서는 subtle border, focus와 disabled 상태를 더 분명하게 조정하겠습니다. 모든 색을 극단적으로 바꾸면 정보 계층이 사라질 수 있어 상태별로 필요한 차이만 강화합니다. 지원하지 않는 환경의 기본값과 `forced-colors`가 함께 적용되는 조합도 실제 browser에서 테스트합니다.

## 장점

- subtle UI의 경계와 상태를 더 명확히 만든다.

## 단점

- more 모드에서 모든 색을 극단적으로 바꾸면 정보 계층이 사라진다.

## 주의사항 / 실무 팁

- more에서 border·focus·disabled 상태를 직접 검수한다.
