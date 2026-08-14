# CSS @starting-style 정리

## 핵심 요약

- 삽입되는 UI의 등장 transition을 CSS만으로 다룰 수 있다.
- 지원하지 않는 브라우저에서는 즉시 나타난다.
- 등장과 퇴장을 다른 상태로 나누어 테스트한다.

## 개념 설명

`@starting-style`은 display 전환이나 새로 삽입된 요소의 시작 스타일을 CSS에 적어 transition이 가능하게 하는 규칙이다.

브라우저는 요소가 렌더 트리에 처음 들어오는 순간의 스타일을 알기 어려운데, `@starting-style`이 그 시작값을 제공한다.

## 예시

```css
.toast {
  opacity: 1;
  transition: opacity .2s;
}
@starting-style { .toast { opacity: 0; } }
```

새 토스트가 DOM에 들어올 때 0에서 1로 자연스럽게 나타난다. 닫힘 애니메이션은 별도 상태가 필요하다.

## 면접 답변 예시

> `@starting-style`은 새로 rendering되는 요소가 transition을 시작할 값을 CSS에 제공하는 규칙입니다. Toast, popover와 dialog의 등장 effect에서 JavaScript로 강제 reflow하는 코드를 줄일 수 있습니다. 초기값과 최종 style, transition property가 맞아야 하며 퇴장과 `display` 전환은 별도 상태와 discrete transition 지원을 검토해야 합니다. Reduced motion에서는 등장 effect를 제거해도 내용과 focus가 정상적으로 제공되게 합니다.

## 장점

- JS로 강제 reflow를 만드는 패턴을 줄인다.

## 단점

- display 전환과 함께 쓰면 종료 상태 관리가 필요하다.

## 주의사항 / 실무 팁

- `@supports` fallback을 확인한다.
