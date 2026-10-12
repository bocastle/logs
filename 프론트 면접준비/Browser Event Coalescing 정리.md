# Browser Event Coalescing 정리

## 핵심 요약

- 고빈도 입력에서도 이벤트 처리 비용을 줄일 수 있다.
- 모든 이벤트 타입에서 coalescing 정보를 주지는 않는다.
- 드로잉처럼 경로 정밀도가 필요한 UI에만 쓴다.

## 개념 설명

Browser Event Coalescing은 브라우저가 높은 빈도의 입력 이벤트를 한 이벤트로 합쳐 메인 스레드 부담을 줄이는 동작이다.

pointer 이벤트에서는 `getCoalescedEvents()`로 합쳐진 원본 움직임을 읽을 수 있다. 드로잉 앱처럼 세밀한 경로가 필요할 때만 사용한다.

## 예시

```js
canvas.addEventListener("pointermove", (event) => {
  const coalesced = event.getCoalescedEvents?.() ?? [];
  const points = coalesced.length > 0 ? coalesced : [event];
  for (const point of points) draw(point.clientX, point.clientY);
});
```

`pointermove`에서 `getCoalescedEvents()`를 읽어 합쳐진 좌표까지 선으로 연결하는 예다.

## 면접 답변 예시

> Event coalescing은 browser가 고빈도 pointer 움직임을 묶어 main thread event 수를 줄이는 동작입니다. 일반 UI는 대표 `pointermove`만 처리해도 되지만 drawing처럼 경로가 중요한 경우 `getCoalescedEvents()`의 중간 좌표를 사용하겠습니다. API가 없거나 event type이 지원하지 않는 환경에는 현재 event를 fallback으로 처리합니다. 모든 point를 즉시 무거운 작업으로 넘기면 이점이 사라지므로 frame 단위 batch와 입력 latency를 함께 측정합니다.

## 장점

- 펜 입력과 캔버스 드로잉의 경로 품질을 높일 수 있다.

## 단점

- 합쳐진 모든 점을 처리하면 다시 비용이 커질 수 있다.

## 주의사항 / 실무 팁

- `pointermove` handler는 짧게 유지하고 그리기는 batch 처리한다.
