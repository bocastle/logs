# View Transitions API 정리

## 핵심 요약

- View Transitions API는 화면 상태가 바뀔 때 이전 화면과 다음 화면 사이의 전환 애니메이션을 브라우저가 연결해주는 API다.
- 단순 CSS 애니메이션과 달리 DOM 변경 전후의 시각적 상태를 캡처해서 자연스럽게 이어준다.
- SPA 내부 전환뿐 아니라 일부 브라우저에서는 문서 간 전환에도 사용할 수 있다.
- 애니메이션보다 중요한 것은 상태 변경 타이밍, 접근성, 성능이다.

## 개념 설명

일반적인 화면 전환은 DOM을 바꾼 뒤 CSS로 새 요소에 애니메이션을 주는 방식이 많다. 이 방식은 이전 화면과 다음 화면의 연결감을 만들기 어렵다. View Transitions API는 브라우저가 이전 화면의 스냅샷과 변경 후 화면의 스냅샷을 잡고, 두 상태 사이를 CSS로 제어할 수 있게 한다.

면접에서는 "페이지 이동 애니메이션 API"라고만 말하기보다, DOM 변경을 감싸서 전환의 시작과 끝을 브라우저가 인식하게 한다고 설명하면 좋다.

## 예시

```js
function openDetail(productId) {
  if (!document.startViewTransition) {
    renderDetail(productId);
    return;
  }

  document.startViewTransition(() => {
    renderDetail(productId);
  });
}
```

```css
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 180ms;
}
```

위 예시는 상세 화면 렌더링을 `startViewTransition` 안에서 수행한다. 브라우저는 변경 전후 상태를 알고 있으므로 자연스러운 전환을 만들 수 있다.

## 면접 답변 예시

"View Transitions API는 화면 변경 전후의 상태를 브라우저가 캡처해서 전환 애니메이션을 만들 수 있게 해주는 API입니다. SPA에서는 라우트 변경이나 리스트에서 상세로 이동하는 상황에 사용할 수 있습니다. 다만 애니메이션이 길거나 복잡하면 오히려 사용성을 해칠 수 있고, `prefers-reduced-motion` 같은 접근성 설정도 고려해야 합니다."

## 장점

- 화면 전환이 끊기지 않고 자연스럽게 보인다.
- 이전 DOM과 다음 DOM을 수동으로 동시에 관리하는 부담이 줄어든다.
- 라우트 전환, 카드 확장, 목록-상세 이동 같은 UI에 잘 맞는다.
- CSS pseudo-element를 통해 전환 스타일을 분리할 수 있다.

## 단점

- 지원 브라우저를 확인해야 한다.
- 복잡한 레이아웃에서는 의도하지 않은 캡처나 깜빡임이 생길 수 있다.
- 애니메이션이 길면 빠른 인터랙션을 방해할 수 있다.
- 상태 변경 로직과 렌더링 타이밍을 잘못 묶으면 버그가 생긴다.

## 주의사항 / 실무 팁

- 지원하지 않는 브라우저에서는 즉시 렌더링으로 fallback한다.
- `prefers-reduced-motion`을 존중해서 모션을 줄일 수 있게 한다.
- 전환 대상 요소에 안정적인 `view-transition-name`을 부여한다.
- 데이터 로딩이 끝나기 전 무리하게 전환하지 않는다.
- 전환 효과는 짧고 명확하게 유지한다.
