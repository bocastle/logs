# DOM Event cancelable과 passive 정리

## 핵심 요약

- 스크롤 성능과 제스처 제어 책임을 분리할 수 있다.
- passive listener에서 `preventDefault()`를 호출해도 기본 동작은 막히지 않는다.
- scroll, wheel, touch listener는 passive 가능 여부를 먼저 분류한다.

## 개념 설명

DOM 이벤트의 `cancelable`과 passive listener 설정은 기본 동작을 막을 수 있는지와 스크롤 차단 여부를 결정한다.

`passive: true` listener에서는 `preventDefault()`가 동작하지 않으므로, 먼저 `event.cancelable`을 확인하고 스크롤을 막아야 하는 제스처만 non-passive로 둔다.

## 예시

```js
window.addEventListener("touchmove", (event) => {
  if (event.cancelable && shouldBlockGesture(event)) event.preventDefault();
}, { passive: false });
```

`cancelable`을 확인한 뒤 필요한 터치 제스처에서만 `preventDefault()`를 호출하는 예다.

## 면접 답변 예시

> Event의 `cancelable`은 기본 동작을 막을 수 있는지, passive option은 listener가 `preventDefault()`를 호출하지 않겠다고 browser에 알리는 계약입니다. Scroll과 touch listener는 기본적으로 passive가 가능한지 먼저 나누고 실제 custom gesture 때문에 차단이 필요한 범위만 `{ passive: false }`로 두겠습니다. Non-passive listener 안에서도 `event.cancelable`을 확인해 막을 수 없는 event에 호출하지 않습니다. Performance trace에서 scroll-blocking listener와 handler 시간을 확인하고 CSS `touch-action`으로 해결할 수 있는 gesture인지도 먼저 봅니다.

## 장점

- 불필요한 non-passive listener 경고를 줄인다.

## 단점

- non-passive listener 남용은 스크롤 시작을 지연시킨다.

## 주의사항 / 실무 팁

- 차단이 필요한 경우 `event.cancelable`을 guard로 둔다.
