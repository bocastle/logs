# Web Push와 Notification Permission 정리

## 핵심 요약

- 사용자가 원하는 이벤트를 탭 밖에서도 알릴 수 있다.
- 페이지 진입 즉시 권한을 요청하면 맥락 없는 차단을 유도한다.
- 사용자가 알림 기능을 켠 동작에서만 requestPermission을 호출한다.

## 개념 설명

Web Push의 Notification permission은 사이트가 시스템 알림을 표시하기 전에 사용자의 명시적 선택을 받는 권한 경계다.

알림의 가치와 빈도를 설명한 버튼의 user gesture 안에서 `Notification.requestPermission`을 호출하고 결과가 granted일 때만 Service Worker의 push 구독 단계로 이동한다.

## 예시

```ts
enableAlertsButton.addEventListener("click", async () => {
  const permission = await Notification.requestPermission();
  if (permission === "granted") await subscribeToPush();
  if (permission === "default") showDismissedState();
  if (permission === "denied") showBrowserSettingsHelp();
});
```

user gesture에서 권한을 요청하고 granted, default, denied를 서로 다른 UI 상태로 처리한다. default는 영구 거절로 단정하지 않는다.

## 면접 답변 예시

> Notification permission은 site 진입 즉시 요청하지 않고 사용자가 알림 기능과 빈도를 이해한 뒤 켜는 동작에서 요청하겠습니다. `granted`, `default`, `denied`를 서로 다른 UI 상태로 처리하고 허용된 경우에만 Service Worker push subscription을 만듭니다. `denied`에서는 prompt를 반복하지 않고 browser 설정과 대체 채널을 안내합니다. 잠금 화면에는 알림 내용이 노출될 수 있어 민감한 상세는 넣지 않고 인증된 page에서 보여 줍니다.

## 장점

- 권한 결과에 따라 구독 요청을 불필요하게 만들지 않을 수 있다.

## 단점

- denied 상태에서 반복 호출해도 브라우저 prompt가 다시 열리지 않을 수 있다.

## 주의사항 / 실무 팁

- default는 나중에 다시 선택할 수 있는 닫힘 상태로 표시한다.
