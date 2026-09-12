# Pointer Capture API 정리

## 핵심 요약

- 드래그와 리사이즈 상호작용이 안정적이다.
- 캡처 해제를 놓치면 이후 입력이 이상하게 라우팅된다.
- pointerup과 pointercancel에서 모두 release를 호출한다.

## 개념 설명

Pointer Capture API는 드래그 중 포인터가 요소 밖으로 나가도 특정 요소가 pointer event를 계속 받게 하는 입력 API다.

`setPointerCapture(pointerId)`를 호출하면 해당 pointerId의 move와 up 이벤트가 요소로 전달되고, 종료 시 `releasePointerCapture`로 풀어야 한다.

## 예시

```ts
thumb.addEventListener("pointerdown", (event) => {
  thumb.setPointerCapture(event.pointerId);
});
thumb.addEventListener("pointermove", updateSlider);
function finishDrag(event: PointerEvent) {
  if (thumb.hasPointerCapture(event.pointerId)) {
    thumb.releasePointerCapture(event.pointerId);
  }
}
thumb.addEventListener("pointerup", finishDrag);
thumb.addEventListener("pointercancel", finishDrag);
```

슬라이더 thumb 밖으로 포인터가 나가도 move를 계속 받아 드래그가 끊기지 않게 한다.

## 면접 답변 예시

> Pointer Capture API는 drag 중 pointer가 element 밖으로 나가도 같은 element가 move와 종료 event를 계속 받게 합니다. `pointerdown`에서 해당 ID를 capture하고 `pointerup`뿐 아니라 `pointercancel`에서도 안전하게 해제하겠습니다. Touch 환경에서는 page scroll과 경쟁할 수 있어 필요한 축에만 `touch-action`을 정해야 합니다. Capture는 pointer 입력만 안정화할 뿐 keyboard 대체 조작과 ARIA 상태를 제공하지 않으므로 slider나 resize control의 접근성 동작은 별도로 구현합니다.

## 장점

- 마우스, 펜, 터치를 같은 pointer 모델로 처리할 수 있다.

## 단점

- 스크롤 제스처와 충돌할 수 있다.

## 주의사항 / 실무 팁

- touch-action 설정과 스크롤 충돌을 함께 확인한다.
