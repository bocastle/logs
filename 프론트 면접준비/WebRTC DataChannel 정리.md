# WebRTC DataChannel 정리

## 핵심 요약

- 애플리케이션 데이터를 peer 사이에 낮은 지연으로 보낼 수 있다.
- bufferedAmount를 무시하면 메모리에 송신 데이터가 계속 쌓인다.
- 채널별 데이터 의미에 맞춰 ordered와 재전송 옵션을 정한다.

## 개념 설명

RTCDataChannel은 RTCPeerConnection 위에서 브라우저 간 임의 데이터를 SCTP로 전달하는 양방향 WebRTC 메시지 채널이다.

`createDataChannel`의 ordered와 재전송 옵션으로 신뢰성 요구를 정하고, 큰 송신은 `bufferedAmount`와 bufferedamountlow를 이용해 backpressure를 적용한다.

## 예시

```ts
const channel = pc.createDataChannel("cursor", {
  ordered: false,
  maxRetransmits: 0,
});
channel.bufferedAmountLowThreshold = 64 * 1024;
function sendCursor(bytes: ArrayBuffer) {
  if (channel.bufferedAmount <= channel.bufferedAmountLowThreshold) channel.send(bytes);
}
```

최신 위치만 중요한 cursor 채널은 ordered를 끄고 재전송하지 않는다. bufferedAmount가 임계값보다 클 때는 새 frame을 합치거나 버린다.

## 면접 답변 예시

> WebRTC DataChannel은 peer 사이에서 text나 binary 데이터를 낮은 지연으로 주고받는 채널입니다. Cursor처럼 최신 값만 중요하면 unordered와 제한된 재전송을 선택할 수 있지만, 상태 patch라면 순서가 뒤바뀌어도 안전한지 먼저 봐야 합니다. 송신이 몰릴 때는 `bufferedAmount`와 `bufferedamountlow`로 queue를 제어하고 오래된 cursor frame은 합치거나 버리겠습니다. DataChannel 자체는 영속 저장을 보장하지 않으므로 반드시 남아야 하는 메시지는 server 승인 흐름을 별도로 둡니다.

## 장점

- 메시지 종류별로 순서와 재전송 정책을 선택할 수 있다.

## 단점

- 순서 없는 채널에서 상태 patch를 보내면 이전 값이 나중에 적용될 수 있다.

## 주의사항 / 실무 팁

- bufferedamountlow 이벤트로 송신 재개 시점을 관리한다.
