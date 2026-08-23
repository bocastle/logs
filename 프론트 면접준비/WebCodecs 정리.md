# WebCodecs 정리

## 핵심 요약

- 프레임 단위 편집과 분석 파이프라인을 만들 수 있다.
- codec 문자열 지원이 기기와 브라우저마다 달라 configure가 실패할 수 있다.
- VideoDecoder.isConfigSupported로 구성을 먼저 확인한다.

## 개념 설명

WebCodecs는 압축된 미디어 chunk와 원시 audio·video frame 사이의 인코딩 및 디코딩 단계를 브라우저에서 직접 제어하는 저수준 API다.

VideoDecoder에 codec을 configure하고 timestamp가 있는 EncodedVideoChunk를 decode하면 output callback으로 VideoFrame이 오며 사용 뒤 `VideoFrame.close`에 해당하는 close 호출로 리소스를 해제해야 한다.

## 예시

```ts
const decoder = new VideoDecoder({
  output: (frame) => {
    renderFrame(frame);
    frame.close();
  },
  error: reportDecodeError,
});
decoder.configure({ codec: "vp09.00.10.08" });
decoder.decode(new EncodedVideoChunk(chunkInit));
```

EncodedVideoChunk를 decoder에 넣고 output의 frame을 렌더링한 직후 close한다. decodeQueueSize가 커지면 입력을 늦추는 backpressure가 필요하다.

## 면접 답변 예시

> WebCodecs는 압축된 media chunk와 raw frame 사이의 codec 단계를 낮은 수준에서 직접 제어하는 API입니다. 프레임 분석이나 편집 pipeline에는 유용하지만 container parsing과 audio·video 동기화까지 대신해 주지는 않습니다. 먼저 `isConfigSupported()`로 codec 구성을 확인하고 `decodeQueueSize`가 커지면 입력을 늦춰 backpressure를 걸겠습니다. 출력된 `VideoFrame`은 성공·drop·error 경로에서 소유권을 분명히 해 반드시 `close()`하고 timestamp 순서도 유지해야 합니다.

## 장점

- 컨테이너 처리와 codec 처리를 별도 계층으로 최적화할 수 있다.

## 단점

- VideoFrame을 닫지 않으면 GPU와 디코더 리소스가 빠르게 쌓인다.

## 주의사항 / 실무 팁

- decodeQueueSize 상한을 정해 입력 생산자를 늦춘다.
