# Web Worker Transferable Object 정리

## 핵심 요약

- 대형 바이너리의 메모리 복사와 GC 압력을 줄일 수 있다.
- 전송 직후 원본 buffer가 detached되어 기존 뷰를 사용할 수 없다.
- 전송한 변수를 이후 코드에서 다시 참조하지 않도록 이름과 scope를 좁힌다.

## 개념 설명

Transferable Object는 큰 리소스를 structured clone으로 복사하지 않고 한 실행 컨텍스트에서 다른 컨텍스트로 소유권을 이전하는 객체다.

`postMessage(data, transferList)`의 transfer list에 `ArrayBuffer`를 넣으면 worker가 같은 메모리 리소스를 넘겨받고 송신 측 buffer는 detached 상태가 되어 byteLength가 0이 된다.

## 예시

```ts
const pixels = new Uint8Array(width * height * 4);
worker.postMessage(
  { type: "PROCESS", buffer: pixels.buffer },
  [pixels.buffer],
);
console.assert(pixels.buffer.byteLength === 0); // detached
```

payload의 buffer와 transfer list에 같은 ArrayBuffer를 넣어 소유권을 옮긴다. 전송 뒤 메인 스레드는 pixels를 다시 읽거나 수정해서는 안 된다.

## 면접 답변 예시

> 전송 시점을 기준으로 버퍼 소유권을 명확히 나눌 수 있다. 작은 데이터는 전송 관리 비용이 복사 절감보다 클 수 있다. 복사가 필요한 소비자가 둘 이상이면 전송 전에 명시적으로 사본을 만든다.

## 장점

- OffscreenCanvas나 MessagePort 같은 단일 소유 리소스도 전달할 수 있다.

## 단점

- payload에 포함하지 않은 객체를 transfer list에만 넣으면 수신 측이 리소스를 찾지 못한다.

## 주의사항 / 실무 팁

- TypedArray 자체가 아니라 그 ArrayBuffer를 transfer list에 넣는다.
