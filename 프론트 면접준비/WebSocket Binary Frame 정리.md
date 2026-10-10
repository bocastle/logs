# WebSocket Binary Frame 정리

## 핵심 요약

- 텍스트 인코딩 없이 바이너리 데이터를 전달할 수 있다.
- 길이 검증 없는 frame은 과도한 메모리 할당을 유발할 수 있다.
- 수신 전에 최대 byteLength를 검사한다.

## 개념 설명

WebSocket Binary Frame은 이미지 조각이나 압축된 이벤트처럼 문자열 변환이 불필요한 payload를 ArrayBuffer 또는 Blob으로 전달하는 메시지 형식이다.

클라이언트의 `binaryType`을 `arraybuffer`로 정하고 frame 앞부분에 version과 message type을 둔 schema를 검증한 뒤 DataView로 본문을 읽는다.

## 예시

```ts
socket.binaryType = "arraybuffer";
socket.onmessage = ({ data }) => {
  if (
    !(data instanceof ArrayBuffer) ||
    data.byteLength < 2 || data.byteLength > MAX_FRAME_BYTES
  ) return;
  const view = new DataView(data);
  const version = view.getUint8(0);
  const type = view.getUint8(1);
  if (version !== 1 || !KNOWN_TYPES.has(type)) return;
  decodeFrame({ type, payload: data.slice(2) });
};
```

binaryType을 고정하고 크기, version, type을 확인한 뒤 payload를 해석한다. 서버와 클라이언트가 같은 byte order와 schema를 써야 한다.

## 면접 답변 예시

> WebSocket binary frame은 text encoding 없이 image 조각이나 압축 event를 `ArrayBuffer`로 전달할 수 있습니다. Decode 전에 전체 byte length와 최소 header 길이, version과 message type을 검증해 과도한 할당과 out-of-bounds read를 막겠습니다. 숫자의 byte order와 field 폭을 protocol에 고정하고 server와 client의 schema version을 함께 운영해야 합니다. 알 수 없는 version은 무조건 연결을 끊기보다 frame 폐기와 client 갱신 신호처럼 호환 정책에 따라 처리합니다.

## 장점

- frame header로 형식 버전과 종류를 작게 표현할 수 있다.

## 단점

- endianness와 숫자 폭이 다르면 같은 바이트를 다른 값으로 해석한다.

## 주의사항 / 실무 팁

- header의 byte order와 필드 폭을 프로토콜 문서에 고정한다.
