# requestIdleCallback 정리

## 핵심 요약

- 낮은 우선순위 작업을 main thread 여유 시간에 실행한다.
- callback 실행 시간을 보장하지 않는다.
- `didTimeout`과 `timeRemaining()` 두 경로를 모두 테스트한다.

## 개념 설명

`requestIdleCallback` 은 브라우저가 main thread에 여유가 있다고 판단한 시점에 낮은 우선순위 callback을 실행하도록 예약하는 API다.

callback은 `IdleDeadline`을 받고 `timeRemaining()`으로 현재 idle period의 추정 남은 시간을, `didTimeout`으로 timeout 때문에 실행됐는지 확인한다. handle은 `cancelIdleCallback()`으로 취소한다.

## 예시

```js
const handle = requestIdleCallback((deadline) => {
  if (deadline.didTimeout || deadline.timeRemaining() > 5) flushAnalytics();
}, { timeout: 3000 });
// cancelIdleCallback(handle);
```

idle budget이 충분하거나 3초 timeout이 되었을 때 분석 queue를 비운다.

## 면접 답변 예시

> `requestIdleCallback`은 반드시 끝나야 하는 작업이 아니라 낮은 우선순위 queue를 main thread의 추정 여유 시간에 조금씩 처리하는 API입니다. Callback 실행 시각은 보장되지 않아 중요한 저장이나 network 전송의 유일한 경로로 쓰면 안 됩니다. `timeRemaining()` budget 안에서 작은 chunk만 처리하고 timeout으로 실행된 경우도 한 번에 긴 task를 만들지 않겠습니다. 지원하지 않는 browser에는 같은 chunk·cancel 계약의 scheduler fallback을 두고 route 변경 시 예약을 취소합니다.

## 장점

- cancel handle로 작업이 더 이상 필요 없을 때 제거한다.

## 단점

- `timeRemaining()`은 보장된 deadline이 아니므로 각 작업이 긴 경우 frame을 넘긴다.

## 주의사항 / 실무 팁

- component unmount·route change에서 idle callback을 취소한다.
