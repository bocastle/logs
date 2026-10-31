# Media Source Extensions 정리

## 핵심 요약

- 적응형 bitrate 플레이어가 필요한 구간만 내려받을 수 있다.
- SourceBuffer updating 중 추가 append를 하면 상태 오류가 발생한다.
- MediaSource.isTypeSupported로 MIME codec을 확인한다.

## 개념 설명

Media Source Extensions는 JavaScript가 미디어 세그먼트를 MediaSource의 SourceBuffer에 순차적으로 붙여 하나의 video 재생 timeline을 만드는 스트리밍 API다.

sourceopen 뒤 MIME codec 문자열로 SourceBuffer를 만들고 updating이 false일 때 appendBuffer하며 `updateend`에서 queue의 다음 세그먼트를 처리한다.

## 예시

```ts
if (!MediaSource.isTypeSupported(MIME_CODEC)) {
  throw new Error(`Unsupported media type: ${MIME_CODEC}`);
}
const mediaSource = new MediaSource();
const objectUrl = URL.createObjectURL(mediaSource);
video.src = objectUrl;
mediaSource.addEventListener("sourceopen", () => {
  URL.revokeObjectURL(objectUrl);
  const sourceBuffer = mediaSource.addSourceBuffer(MIME_CODEC);
  sourceBuffer.addEventListener("updateend", () => appendNextSegment(sourceBuffer));
  appendNextSegment(sourceBuffer);
});
```

MediaSource가 열린 뒤 SourceBuffer를 만들고 updateend를 다음 append 시점으로 사용한다. 동시에 appendBuffer를 호출하면 InvalidStateError가 날 수 있다.

## 면접 답변 예시

> Media Source Extensions는 JavaScript가 media segment를 `SourceBuffer`에 순서대로 append해 video timeline을 만드는 API입니다. 먼저 MIME·codec 지원을 확인하고 한 buffer에는 queue를 두어 `updating`이 끝난 `updateend`에서만 다음 segment를 넣겠습니다. 장시간 재생에서는 current time 뒤 안전 구간을 남기고 오래된 buffered range를 remove해 memory와 quota를 관리합니다. Append와 decode error, network 중단 및 bitrate 전환을 같은 state machine에서 처리하고 지원하지 않으면 일반 media source fallback을 제공합니다.

## 장점

- 재생 buffer 범위를 보며 앞뒤 세그먼트를 관리할 수 있다.

## 단점

- codec과 container 조합이 맞지 않으면 세그먼트 파싱이 실패한다.

## 주의사항 / 실무 팁

- append queue를 하나로 직렬화하고 updateend에서만 다음 항목을 꺼낸다.
