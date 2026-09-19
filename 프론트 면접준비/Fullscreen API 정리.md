# Fullscreen API 정리

## 핵심 요약

- 영상, 지도, 프레젠테이션 같은 몰입형 화면을 제공할 수 있다.
- 사용자 제스처 없이 요청하면 실패한다.
- 요청 실패를 catch하고 일반 화면을 유지한다.

## 개념 설명

Fullscreen API는 특정 요소를 브라우저 전체 화면으로 전환하고 종료 상태를 감지하는 API다.

`requestFullscreen`은 사용자 제스처가 필요하고, `fullscreenchange`와 `fullscreenerror` 이벤트로 전환 결과를 확인한다.

## 예시

```ts
try {
  await player.requestFullscreen();
} catch (error) {
  showFullscreenUnavailable(error);
}
document.addEventListener("fullscreenchange", () => {
  setFullscreen(Boolean(document.fullscreenElement));
});
```

전체 화면 여부는 버튼 상태가 아니라 `document.fullscreenElement`를 기준으로 동기화한다.

## 면접 답변 예시

> Fullscreen API는 video나 map element를 browser의 전체 화면으로 전환하는 기능입니다. `requestFullscreen()`은 사용자 동작과 permission policy가 필요해 거절될 수 있으므로 실패를 처리하고 기존 화면을 그대로 유지하겠습니다. Button을 눌렀다는 사실이 아니라 `fullscreenchange`의 `document.fullscreenElement`를 기준으로 UI 상태를 동기화합니다. Escape 종료, focus와 keyboard 조작, iframe policy와 mobile 방향 전환까지 실제 환경에서 확인합니다.

## 장점

- 요소 단위로 전체 화면을 요청할 수 있다.

## 단점

- iframe에서는 allowfullscreen과 정책이 필요하다.

## 주의사항 / 실무 팁

- fullscreenchange에서 UI 상태를 항상 다시 계산한다.
