# WebTransport 정리

## 핵심 요약

- 낮은 지연과 다양한 전송 방식을 한 연결에서 쓸 수 있다.
- 지원 브라우저와 서버 인프라가 제한적이다.
- 데이터별로 stream과 datagram 선택 기준을 문서화한다.

## 개념 설명

WebTransport는 HTTP/3 위에서 양방향 stream과 datagram을 제공하는 저지연 클라이언트-서버 통신 API다.

신뢰성 있는 stream과 손실을 허용하는 datagram을 선택할 수 있고, WebSocket보다 전송 특성을 더 세밀하게 나눌 수 있다.

## 예시

```ts
const transport = new WebTransport("https://example.com/realtime");
await transport.ready;
const writer = transport.datagrams.writable.getWriter();
try {
  await writer.write(encodePosition(position));
} finally {
  writer.releaseLock();
}
```

위치처럼 최신 값만 중요한 데이터는 datagram으로 보내고, 주문 같은 순서와 신뢰성이 필요한 데이터는 stream을 써야 한다.

## 면접 답변 예시

> WebTransport는 HTTP/3 연결에서 reliable stream과 unreliable datagram을 함께 선택할 수 있는 client-server API입니다. 주문처럼 순서와 전달이 필요한 data는 stream, 최신 위치만 중요해 일부 손실을 허용할 수 있는 data는 datagram에 두겠습니다. 지원 browser와 HTTP/3 server·network 경로가 제한될 수 있어 capability 확인과 WebSocket fallback이 필요합니다. `ready`와 `closed` 실패, 재연결을 처리하고 손실률과 latency를 양쪽에서 관찰합니다.

## 장점

- 실시간 게임이나 미디어 제어에 적합한 datagram을 제공한다.

## 단점

- datagram은 손실과 순서 바뀜을 애플리케이션이 감수해야 한다.

## 주의사항 / 실무 팁

- WebSocket fallback을 준비한다.
