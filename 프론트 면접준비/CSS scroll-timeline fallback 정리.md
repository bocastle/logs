# CSS scroll-timeline fallback 정리

## 핵심 요약

- 미지원 환경에서도 정보 손실을 막는다.
- 기본 상태를 숨김으로 두면 콘텐츠가 영원히 안 보일 수 있다.
- 기본 스타일에 읽을 수 있는 최종 상태를 둔다.

## 개념 설명

CSS scroll-timeline fallback은 scroll-driven animation을 지원하지 않거나 motion 감소가 필요한 환경에서도 콘텐츠와 상태를 완전히 보여 주는 대안이다.

기본 CSS에 최종 상태를 놓고 `@supports (animation-timeline: scroll())`안에서만 timeline을 켜며, 필수 상태를 scroll event polyfill에 의존하지 않는다.

## 예시

```css
.reveal { opacity: 1; transform: none; }
@supports (animation-timeline: scroll()) {
  .reveal { animation: reveal linear both; animation-timeline: view(); }
}
```

fallback에서 콘텐츠를 기본적으로 보이고 timeline 지원 환경에서만 reveal 효과를 덧붙인다.

## 면접 답변 예시

> Scroll timeline은 scroll 진행률을 animation 시간축으로 쓰는 기능이지만 핵심 정보가 이 효과에 의존하면 안 됩니다. 기본 CSS에는 opacity와 transform의 읽을 수 있는 최종 상태를 두고 실제로 사용하는 문법의 `@supports` 안에서만 animation을 추가하겠습니다. `prefers-reduced-motion: reduce`에서는 timeline 지원 여부와 무관하게 움직임을 제거하고 JS scroll handler polyfill은 필수 동작에 사용하지 않습니다. 지원·미지원, reduced motion, keyboard scroll과 긴 page에서 콘텐츠 노출과 main-thread 비용을 각각 확인합니다.

## 장점

- progressive enhancement 구조가 CSS에 드러난다.

## 단점

- 기능 탐지 조건이 실제 사용 문법과 다르면 헛탐지가 생긴다.

## 주의사항 / 실무 팁

- `@supports`의 조건을 실제 `animation-timeline` 선언과 맞춘다.
