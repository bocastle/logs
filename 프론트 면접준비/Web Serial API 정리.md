# Web Serial API 정리

## 핵심 요약

- 마이크로컨트롤러와 계측 장비의 console을 웹으로 제공할 수 있다.
- 잘못된 baudRate와 parity 설정은 읽을 수 없는 바이트를 만든다.
- 장치 protocol의 delimiter나 length prefix를 누적 buffer에서 해석한다.

## 개념 설명

Web Serial API는 사용자가 선택한 직렬 포트를 ReadableStream과 WritableStream으로 열어 텍스트 또는 binary protocol을 송수신하게 하는 API다.

`navigator.serial.requestPort` 뒤 장치와 맞는 `baudRate`로 open하고 readable의 reader lock을 얻어 반복 read하며 종료 시 lock을 풀고 port를 닫는다.

## 예시

```ts
const port = await navigator.serial.requestPort({
  filters: [{ usbVendorId: 0x2341 }],
});
await port.open({ baudRate: 115200 });
if (!port.readable) throw new Error("Serial readable stream is unavailable");
const reader = port.readable.getReader();
try {
  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    parseSerialBytes(value);
  }
} finally {
  reader.releaseLock();
  await port.close();
}
```

navigator.serial로 선택한 port를 115200 baudRate로 열고 readable stream을 순차 소비한다. 메시지 경계는 read 호출 경계와 일치하지 않으므로 별도 parser가 필요하다.

## 면접 답변 예시

> Web Serial은 사용자가 선택한 serial port를 Streams API로 읽고 쓰게 하는 기능입니다. Device protocol에 맞는 baud rate와 parity를 사용하고 read chunk가 message boundary와 일치한다고 가정하지 않고 누적 parser에서 delimiter나 length prefix를 처리하겠습니다. 종료할 때는 read loop를 cancel한 뒤 reader·writer lock을 풀고 port를 닫아야 close가 대기하지 않습니다. Disconnect, malformed packet과 reconnect를 test하고 broad device filter와 수신 data의 민감성도 검토합니다.

## 장점

- Streams API로 수신 backpressure와 변환 파이프를 구성할 수 있다.

## 단점

- 한 packet이 여러 read로 나뉘거나 여러 packet이 한 read에 합쳐질 수 있다.

## 주의사항 / 실무 팁

- AbortSignal이나 reader.cancel로 읽기 loop를 먼저 끝낸다.
