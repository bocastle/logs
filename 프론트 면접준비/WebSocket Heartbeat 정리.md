# WebSocket Heartbeat 정리

## 핵심 요약

- close 이벤트가 늦는 죽은 연결을 더 빠르게 감지할 수 있다.
- 주기가 너무 짧으면 서버, 네트워크, 배터리 비용이 커진다.
- ping id와 전송 시각을 pong에서 대조한다.

## 개념 설명

WebSocket heartbeat는 애플리케이션 수준의 ping을 주기적으로 보내고 정해진 시간 안의 pong 응답으로 프록시와 모바일 네트워크의 반쪽 연결을 감지하는 생존 확인 프로토콜이다.

브라우저 JavaScript는 프로토콜 제어 frame을 직접 보내지 못하므로 JSON ping에 id를 넣고 대응하는 pong이 timeout 전에 오지 않으면 소켓을 닫아 재연결을 시작한다.

## 예시

```ts
let pendingPing: { id: string; timeout: number } | null = null;
function heartbeat() {
  if (pendingPing || socket.readyState !== WebSocket.OPEN) return;
  const id = crypto.randomUUID();
  socket.send(JSON.stringify({ type: "ping", id }));
  const timeout = window.setTimeout(
    () => socket.close(4000, "heartbeat timeout"),
    10_000,
  );
  pendingPing = { id, timeout };
}
function onMessage(message: Message) {
  if (message.type === "pong" && message.id === pendingPing?.id) {
    clearTimeout(pendingPing.timeout);
    pendingPing = null;
  }
}
```

각 ping 뒤 pong timeout을 예약하고 응답에서 해제한다. 실제 구현은 ping id를 비교해 이전 주기의 늦은 pong이 현재 timeout을 취소하지 못하게 해야 한다.

## 면접 답변 예시

> WebSocket heartbeat는 application ping과 pong으로 close event가 늦게 오는 half-open 연결을 더 빨리 찾는 방법입니다. Browser JavaScript는 protocol ping frame을 직접 보내지 못하므로 message에 ping ID를 넣고 같은 ID의 pong만 현재 timeout을 해제하게 하겠습니다. 주기는 proxy idle timeout보다 짧되 server와 battery 비용을 고려하고, background tab의 timer throttling 때문에 정상 연결을 끊지 않도록 visibility 정책도 둡니다. Heartbeat timeout close code와 round-trip time을 일반 server 종료와 분리해 기록합니다.

## 장점

- 왕복 지연을 연결 건강 지표로 기록할 수 있다.

## 단점

- 모든 pong에서 timeout을 지우면 늦은 응답이 새 검사 실패를 숨길 수 있다.

## 주의사항 / 실무 팁

- visibility 상태와 서버 idle timeout을 고려해 주기를 정한다.
