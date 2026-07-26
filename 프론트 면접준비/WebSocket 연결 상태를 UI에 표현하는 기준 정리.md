# WebSocket 연결 상태를 UI에 표현하는 기준 정리

## 핵심 요약

- 사용자가 메시지 지연과 전송 가능 여부를 판단할 수 있다.
- open 이벤트만으로 connected를 유지하면 끊어진 반쪽 연결을 놓친다.
- 상태와 함께 마지막 정상 수신 시각을 기록한다.

## 개념 설명

WebSocket 연결 상태 UI는 소켓의 readyState를 그대로 노출하지 않고 사용자가 할 수 있는 행동 기준으로 connecting, connected, stale, offline 상태를 표현하는 방식이다.

open은 connected로, heartbeat 기한 초과는 stale로, close 뒤 재연결 실패는 offline으로 전환하고 송신 버튼과 마지막 수신 시각을 상태에 맞춰 갱신한다.

## 예시

```ts
type ConnectionState = "connecting" | "connected" | "stale" | "offline";
function renderConnection(state: ConnectionState) {
  status.textContent = {
    connecting: "연결 중", connected: "실시간 연결됨",
    stale: "응답 확인 중", offline: "오프라인",
  }[state];
  sendButton.disabled = state !== "connected";
}
```

연결 준비와 응답 지연을 같은 실패로 표시하지 않는다. stale 동안에는 재확인 중임을 알리고 중복 송신을 막는다.

## 면접 답변 예시

> 상태 전이를 로그와 UI에서 같은 enum으로 추적할 수 있다. offline인데 입력을 받아 버리면 전송되지 않은 작업이 조용히 유실된다. 오프라인 송신을 막거나 durable outbox에 저장한다는 정책을 명시한다.

## 장점

- 재연결 중인 화면이 멈춘 것으로 오해되는 일을 줄인다.

## 단점

- 짧은 네트워크 흔들림마다 큰 경고를 띄우면 사용자를 방해한다.

## 주의사항 / 실무 팁

- 짧은 stale 상태에는 비차단 표시를 쓰고 장기 실패에서만 행동을 요구한다.
