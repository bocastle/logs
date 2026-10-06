# Declarative Event Listener 패턴 정리

## 핵심 요약

- 스크롤 성능과 이벤트 의도를 브라우저에 알려줄 수 있다.
- passive 리스너에서 취소를 기대하면 버그가 생긴다.
- preventDefault 전에는 cancelable을 확인한다.

## 개념 설명

DOM 이벤트 처리에서 cancelable과 passive는 기본 동작 취소 가능 여부와 리스너가 스크롤을 막지 않겠다는 계약을 표현한다.

`event.cancelable`이 false이면 `preventDefault`가 효과 없고, passive listener에서는 `preventDefault`를 호출해도 브라우저가 무시할 수 있다.

## 예시

```ts
const controller = new AbortController();
element.addEventListener("touchmove", onMove, {
  passive: true,
  signal: controller.signal,
});
form.addEventListener("submit", onSubmit, { signal: controller.signal });

function onSubmit(event: Event) {
  if (event.cancelable) event.preventDefault();
}

function disposeListeners() {
  controller.abort();
}
```

스크롤을 막지 않는 move 이벤트는 passive로 두고, 취소 가능한 이벤트에서만 preventDefault를 호출한다.

## 면접 답변 예시

> Event listener option은 passive, once와 signal로 기본 동작과 수명을 선언하는 계약입니다. Scroll을 막지 않는 listener는 passive로 두고 취소가 필요한 event만 non-passive로 분리하며 `preventDefault()` 전에는 cancelable 여부를 확인하겠습니다. 여러 listener에 같은 `AbortSignal`을 전달하면 component cleanup에서 한 번에 제거해 중복 등록과 memory leak을 줄일 수 있습니다. 다만 기본 동작을 막는 것이 keyboard와 form 접근성 흐름까지 끊지 않는지도 함께 테스트합니다.

## 장점

- 이벤트 취소 가능성을 코드에서 명확히 확인한다.

## 단점

- 기본 동작 취소가 접근성 흐름을 막을 수 있다.

## 주의사항 / 실무 팁

- 스크롤 관련 이벤트는 passive 기본값을 검토한다.
