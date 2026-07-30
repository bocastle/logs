# CSS anchor positioning 도입 전에 확인할 것 정리

## 핵심 요약

- floating UI 위치 계산 JS를 줄일 수 있다.
- 미지원 환경에서 메뉴가 잘못된 위치에 나올 수 있다.
- 대표 viewport 모서리와 중첩 scroll container를 테스트한다.

## 개념 설명

CSS Anchor Positioning 도입 검토는 trigger와 부유 UI의 연결, viewport 가장자리 fallback, scroll container, 지원 범위를 현재 JS 배치 요구와 비교하는 작업이다.

anchor에 `anchor-name`을 주고 positioned element에 `position-anchor`와 `anchor()`를 연결한다. 여유 공간이 없을 때의 `position-try-fallbacks`와 미지원 환경의 absolute/JS fallback을 함께 정한다.

## 예시

```css
.trigger { anchor-name: --menu-trigger; }
.menu {
  position: fixed;
  position-anchor: --menu-trigger;
  inset-block-start: anchor(bottom);
  position-try-fallbacks: flip-block;
}
```

메뉴를 trigger 아래에 배치하고 하단 공간이 부족하면 block 방향을 뒤집는 예다.

## 면접 답변 예시

> fallback 배치 정책을 선언적으로 남긴다. 복잡한 collision·arrow·virtual anchor 요구는 JS가 더 적합할 수 있다. tooltip보다 menu·combobox처럼 실제 컴포넌트에서 평가한다.

## 장점

- anchor 크기와 위치 변경을 CSS 레이아웃에 위임한다.

## 단점

- overflow 조상과 top layer 여부에 따라 예상과 다를 수 있다.

## 주의사항 / 실무 팁

- `@supports (position-anchor: --x)`로 fallback을 분리한다.
