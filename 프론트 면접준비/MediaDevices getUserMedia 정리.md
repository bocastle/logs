# MediaDevices getUserMedia 정리

## 핵심 요약

- 브라우저에서 화상 회의와 녹음 기능을 구현할 수 있다.
- 권한 거절과 장치 없음 상태가 흔하다.
- 권한 요청 전에 카메라와 마이크가 필요한 이유를 설명한다.

## 개념 설명

MediaDevices getUserMedia는 사용자의 카메라나 마이크 접근 권한을 요청하고 `MediaStream`을 반환하는 브라우저 API다.

`navigator.mediaDevices.getUserMedia`는 secure context와 사용자 허용이 필요하며, constraint를 만족하지 못하면 권한 허용 전후 모두 실패할 수 있다.

## 예시

```ts
const stream = await navigator.mediaDevices.getUserMedia({
  video: { width: { ideal: 1280 } },
  audio: true,
});
video.srcObject = stream;
```

카메라와 마이크 stream을 video 요소에 연결한다. 종료 시에는 track을 stop해야 장치 표시와 배터리 사용을 끝낼 수 있다.

## 면접 답변 예시

> `getUserMedia()`는 browser에서 camera와 microphone 권한을 요청하고 `MediaStream`을 받는 API입니다. 권한 prompt부터 띄우기보다 사용자가 통화나 녹화를 시작하는 맥락에서 필요한 이유를 먼저 설명하겠습니다. `NotAllowedError`, `NotFoundError`와 constraint 실패를 구분해 다시 시도할 수 있는 안내를 주고, 권한 전에는 device label이 제한될 수 있다는 점도 고려합니다. 종료 버튼과 페이지 이탈 시 모든 track을 `stop()`해 장치 표시와 배터리 사용이 남지 않게 합니다.

## 장점

- constraint로 해상도와 장치 요구를 표현할 수 있다.

## 단점

- 활성 track을 정리하지 않으면 카메라가 계속 켜져 보인다.

## 주의사항 / 실무 팁

- 페이지 이탈과 종료 버튼에서 모든 track을 stop한다.
