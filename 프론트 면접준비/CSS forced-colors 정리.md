# CSS forced-colors 정리

## 핵심 요약

- 고대비 사용자에게 상태 경계를 제공한다.
- 색만으로 구분한 상태는 forced palette에서 같아질 수 있다.
- Windows High Contrast와 DevTools emulation에서 테스트한다.

## 개념 설명

CSS `forced-colors` media feature는 사용자 에이전트가 작성자 color를 제한된 system color palette로 교체하는 고대비 환경이 활성인지 알린다.

`@media (forced-colors: active)`에서 `Canvas`, `CanvasText`, `ButtonText`, `Highlight`같은 system color를 사용한다. `forced-color-adjust: none`은 자동 조정을 막으므로 정보 손실을 막는 작은 영역에만 쓴다.

## 예시

```css
@media (forced-colors: active) {
  .selected { border: 2px solid Highlight; }
  .icon { fill: currentColor; }
}
```

색 면 대신 Highlight border와 `currentColor` icon으로 선택 상태를 유지한다.

## 면접 답변 예시

> `forced-colors`는 browser가 작성자 색을 사용자의 제한된 system palette로 바꾸는 고대비 환경을 감지하는 media feature입니다. 이때 background image나 shadow만으로 표시한 선택 상태가 사라질 수 있어 system color의 border, text와 `currentColor` icon으로 의미를 남기겠습니다. `forced-color-adjust: none`은 자동 조정을 막아야 정보가 보존되는 작은 영역에만 사용하고 그 영역의 대비를 직접 책임져야 합니다. Windows High Contrast와 emulation에서 focus, selected, disabled 상태를 모두 확인합니다.

## 장점

- OS system color palette와 UI를 일치시킨다.

## 단점

- `forced-color-adjust: none` 남용은 사용자 선호를 무시한다.

## 주의사항 / 실무 팁

- system color와 border·text 상태를 우선한다.
