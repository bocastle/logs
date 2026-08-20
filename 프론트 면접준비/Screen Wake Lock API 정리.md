# Screen Wake Lock API 정리

## 핵심 요약

- 레시피, 운동, 지도처럼 계속 봐야 하는 화면에 유용하다.
- 배터리 사용량이 늘어 사용자 불만이 생길 수 있다.
- 사용자가 명시적으로 켠 모드에서만 요청한다.

## 개념 설명

Screen Wake Lock API는 사용자가 보고 있는 동안 화면이 자동으로 꺼지지 않도록 브라우저에 요청하는 API다.

`navigator.wakeLock.request('screen')`은 배터리, 탭 visibility, 권한 정책에 따라 해제될 수 있으므로 release 이벤트를 다뤄야 한다.

## 예시

```ts
let lock: WakeLockSentinel | null = null;
let userEnabled = false;

async function requestWakeLock() {
  if (!userEnabled || document.visibilityState !== "visible") return;
  try {
    lock = await navigator.wakeLock.request("screen");
    lock.addEventListener("release", () => markWakeLockOff());
  } catch {
    markWakeLockOff();
  }
}

document.addEventListener("visibilitychange", async () => {
  if (document.visibilityState === "visible") await requestWakeLock();
});
```

문서가 숨겨지면 wake lock이 풀릴 수 있어 visible 복귀 시 다시 요청하는 흐름을 둔다.

## 면접 답변 예시

> Screen Wake Lock API는 레시피나 운동 화면처럼 사용자가 계속 보고 있어야 할 때 화면 꺼짐을 막아 달라고 요청하는 기능입니다. 배터리와 OS 정책 때문에 요청이 거절되거나 탭이 숨겨질 때 자동으로 해제될 수 있으므로 영구 보장으로 다루면 안 됩니다. 사용자가 명시적으로 켠 모드에서만 요청하고 release 상태를 UI에 반영하겠습니다. Visible 상태로 돌아왔을 때도 그 사용자 설정이 아직 켜져 있을 때만 재요청하고, 미지원·절전 환경에는 대체 안내를 제공합니다.

## 장점

- 사용자 조작 없이 화면 유지 의도를 표현할 수 있다.

## 단점

- 브라우저와 OS 정책에 따라 요청이 거절될 수 있다.

## 주의사항 / 실무 팁

- release 이벤트와 visibilitychange를 함께 처리한다.
