# CSS prefers-reduced-motion 정리

## 핵심 요약

- 시스템 접근성 선호를 CSS에 직접 반영한다.
- media query는 JS·canvas motion을 자동으로 바꾸지 않는다.
- 애니메이션 종류별 대체 상태를 정한다.

## 개념 설명

CSS `prefers-reduced-motion` media feature는 사용자가 운영체제에서 비필수 움직임 감소를 요청했는지 스타일시트에서 탐지한다.

`@media (prefers-reduced-motion: reduce)`에서 animation·transition·smooth scrolling을 제거하거나 덜 자극적인 효과로 바꾸고, no-preference를 기본 가정으로 둔다.

## 예시

```css
.modal { transition: opacity 180ms; }
@media (prefers-reduced-motion: reduce) {
  .modal { transition: none; }
}
```

모달의 큰 등장 motion을 reduce 환경에서 즉시 표시로 바꾸는 기본 문법이다.

## 면접 답변 예시

> `prefers-reduced-motion`은 사용자가 OS에서 비필수 움직임을 줄여 달라고 설정했는지 CSS에서 확인하는 media feature입니다. Reduce 환경에서는 큰 이동과 parallax, smooth scrolling을 없애거나 opacity처럼 덜 자극적인 전환으로 바꾸겠습니다. 모든 animation을 universal selector로 강제로 끄면 progress나 상태 전달까지 사라질 수 있어 component별 motion 목적에 맞는 대체 표현이 필요합니다. CSS 밖의 canvas와 JavaScript animation도 같은 media query를 구독하고 DevTools와 실제 OS 설정에서 전환을 검증합니다.

## 장점

- 설정이 켜진 사용자에게만 대체 스타일을 제공한다.

## 단점

- 상태 정보가 animation에만 있으면 reduce에서 손실된다.

## 주의사항 / 실무 팁

- reduce query를 component motion token 가까이 둔다.
