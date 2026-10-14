# Off-main-thread UI 전략 정리

## 핵심 요약

- 입력과 스크롤 반응성을 유지하기 쉽다.
- 메시지 복사 비용이 계산 이점을 삼킬 수 있다.
- long task가 반복되는 작업부터 후보로 삼는다.

## 개념 설명

Off-main-thread UI 전략은 입력 반응성을 지키기 위해 계산, 파싱, 렌더링 일부를 메인 스레드 밖으로 옮기는 설계다.

Web Worker, Worklet, OffscreenCanvas, chunking을 조합하되 DOM 접근은 메인 스레드에 남는다. 메시지 직렬화와 transfer 비용도 성능 예산에 포함해야 한다.

## 예시

```ts
const bytes = new Uint8Array(rawData);
worker.postMessage(
  { type: "parse", buffer: bytes.buffer },
  [bytes.buffer],
);
```

ArrayBuffer를 transfer하면 큰 data 복사를 줄일 수 있지만 송신 측의 buffer는 detach되므로 이후 다시 읽지 않는 소유권 이동이어야 한다.

## 면접 답변 예시

> Off-main-thread 전략은 반복되는 long task의 계산·parsing·image 처리를 worker로 옮겨 input과 scroll 반응성을 지키는 방법입니다. DOM 접근은 main thread에 남고 message serialization과 result 반영 비용이 생기므로 계산이 충분히 큰 작업부터 profile하겠습니다. Transferable은 복사를 줄이지만 sender buffer가 detach되는 소유권 변화도 고려해야 합니다. Request ID와 cancel message를 protocol에 넣어 오래된 worker 결과가 최신 UI를 덮지 않게 하고 INP와 total work를 함께 비교합니다.

## 장점

- 무거운 파싱과 이미지 처리를 UI 렌더링과 분리한다.

## 단점

- DOM 접근이 필요한 작업은 worker로 바로 옮길 수 없다.

## 주의사항 / 실무 팁

- transferable object로 복사 비용을 줄인다.
