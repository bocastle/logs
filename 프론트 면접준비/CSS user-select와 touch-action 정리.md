# CSS user-select와 touch-action 정리

## 핵심 요약

- 드래그와 스와이프 중 의도치 않은 선택을 줄인다.
- `touch-action: none`은 확대와 스크롤 기대를 막을 수 있다.
- 텍스트 복사가 필요한 영역에는 `user-select`를 유지한다.

## 개념 설명

`user-select`와 `touch-action`은 텍스트 선택과 터치 gesture의 브라우저 기본 동작을 조절하는 상호작용 CSS 속성이다.

`user-select: none`은 드래그 중 텍스트 선택을 줄이고, `touch-action`은 pan, pinch-zoom 같은 gesture를 브라우저가 처리할지 결정한다.

## 예시

```css
.drag-handle {
  user-select: none;
  touch-action: pan-y;
}
```

세로 스크롤 gesture는 유지하면서 드래그 핸들 텍스트 선택을 줄이는 예다.

## 면접 답변 예시

> `user-select`는 text 선택을, `touch-action`은 browser가 pan과 zoom 같은 touch gesture를 맡을 범위를 정합니다. Drag handle에서 의도치 않은 text 선택만 막고 세로 page scroll이 필요하다면 `touch-action: pan-y`처럼 필요한 축은 유지하겠습니다. `touch-action: none`을 넓게 적용하면 scroll과 pinch zoom까지 막아 사용성을 해칠 수 있습니다. Custom gesture에는 keyboard 대체 조작과 명확한 state를 제공하고 touch, mouse와 keyboard를 모두 테스트합니다.

## 장점

- 브라우저 스크롤과 커스텀 gesture의 책임을 나눌 수 있다.

## 단점

- `user-select: none` 남용은 복사 가능한 텍스트 접근성을 해친다.

## 주의사항 / 실무 팁

- touch-action은 필요한 축만 제한한다.
