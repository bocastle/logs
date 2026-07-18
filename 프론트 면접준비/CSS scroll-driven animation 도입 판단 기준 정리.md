# CSS scroll-driven animation 도입 판단 기준 정리

## 핵심 요약

- Scroll-driven animation은 시간 대신 scroll 또는 view progress를 CSS animation timeline에 연결한다.
- 문서 진행률처럼 정보 이해를 돕는 경우에 우선 사용하고 장식용 parallax는 접근성과 성능을 따로 검토한다.
- `prefers-reduced-motion`과 미지원 브라우저에서도 정보와 조작이 유지되는 fallback을 먼저 정한다.

## 개념 설명

CSS scroll-driven animation은 애니메이션 진행도를 시간 대신 스크롤 위치에 연결한다. 전체 문서 진행률은 `scroll()` timeline, 요소가 뷰포트에 들어오고 나가는 구간은 `view()` timeline으로 표현할 수 있다.

JavaScript scroll handler에서 매 프레임 style을 바꾸는 코드를 줄일 수 있지만 브라우저가 모든 효과를 항상 compositor에서 처리한다는 뜻은 아니다. Layout과 paint가 큰 속성은 여전히 비용이 들 수 있다. Timeline의 기준 scroller와 진행 범위를 명확히 하고 실제 프레임을 확인한다.

## 예시

```css
.progress {
  animation: grow linear both;
  animation-timeline: scroll(root block);
}
@media (prefers-reduced-motion: reduce) { .progress { animation: none; } }
```

문서 진행률 bar를 root scroller에 연결한 예다. Reduced motion에서 animation을 없앤다면 진행 정보가 사라지는지 확인하고, 필요하면 숫자나 다른 정적 단서를 제공한다.

## 면접 답변 예시

> Scroll-driven animation은 시간 대신 scroll이나 view progress를 CSS animation timeline에 연결하는 기능입니다. 문서 진행률처럼 스크롤 위치가 정보 자체인 경우에는 유용하지만 장식 모션에 무조건 적용하지 않겠습니다. Timeline의 기준 scroller와 진행 범위를 정하고, reduced motion과 미지원 환경에서도 정보가 남도록 fallback을 둡니다. DevTools에서 layout·paint 비용과 프레임 안정성을 실제로 확인합니다.

## 장점

- 스크롤 진행도와 시각 상태를 CSS에서 선언적으로 연결할 수 있다.
- JavaScript의 고빈도 scroll handler와 상태 동기화 코드를 줄일 수 있다.

## 단점

- 기준 scroller나 timeline 범위가 잘못되면 애니메이션이 시작되지 않거나 끝까지 진행되지 않는다.
- Layout과 paint가 비싼 속성을 움직이면 선언형 API여도 프레임이 불안정할 수 있다.

## 주의사항 / 실무 팁

- `prefers-reduced-motion`과 미지원 fallback에서 같은 정보와 조작이 유지되는지 먼저 정한다.
- Nested scroller, 짧은 문서와 동적 콘텐츠 추가 뒤 timeline 범위를 테스트한다.
