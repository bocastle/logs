# WebGPU 개요 정리

## 핵심 요약

- 복잡한 시각화와 3D 렌더링을 브라우저에서 처리할 수 있다.
- adapter나 device 요청이 실패하면 GPU 화면을 시작할 수 없다.
- navigator.gpu와 requestAdapter 결과를 확인한 뒤 Canvas 2D나 서버 렌더링 경로로 전환한다.

## 개념 설명

WebGPU는 브라우저에서 GPU를 사용해 그래픽 렌더링이나 병렬 계산을 수행하는 저수준 그래픽 API다.

WebGPU는 adapter, device, pipeline, command encoder를 명시적으로 구성하고 command buffer를 queue에 제출한다.

## 예시

```ts
async function renderWithWebGPU() {
  if (!("gpu" in navigator)) {
    renderCanvasFallback();
    return;
  }
  const adapter = await navigator.gpu.requestAdapter();
  if (!adapter) {
    renderCanvasFallback();
    return;
  }
  const device = await adapter.requestDevice();
  const encoder = device.createCommandEncoder();
  // render pass -> pipeline -> draw -> submit
  device.queue.submit([encoder.finish()]);
}
```

WebGPU는 command를 기록한 뒤 queue에 제출한다. adapter나 device를 얻지 못하면 같은 데이터의 Canvas 2D 정적 렌더링이나 서버 렌더링 화면으로 전환한다.

## 면접 답변 예시

> WebGPU는 browser에서 GPU rendering과 병렬 계산을 세밀하게 제어하는 저수준 API입니다. Adapter와 device, pipeline과 command buffer를 직접 구성하므로 복잡한 시각화에 강하지만 shader와 resource lifetime까지 application이 책임져야 합니다. 시작할 때 `navigator.gpu`와 adapter 결과를 확인하고 지원하지 않으면 Canvas 2D나 server-rendered 결과로 전환하겠습니다. 실행 중 `device.lost`가 발생할 수 있으므로 resource를 다시 만들거나 fallback으로 내려가는 복구 흐름과 frame time·GPU memory를 함께 검증합니다.

## 장점

- GPU 병렬성을 활용해 CPU 부담을 줄일 수 있다.

## 단점

- 리소스 해제와 device loss 처리가 어렵다.

## 주의사항 / 실무 팁

- 프레임 시간, draw call, GPU memory를 함께 관찰한다.
