# WebGL 렌더링 파이프라인 정리

## 핵심 요약

- 병목을 CPU 제출·vertex·fragment·texture bandwidth로 나눈다.
- `readPixels`·query 결과 대기는 CPU/GPU 동기화 stall을 만들 수 있다.
- WebGL profiler와 frame capture로 CPU·GPU 시간을 분리한다.

## 개념 설명

WebGL rendering pipeline에서는 JavaScript가 buffer·texture·program·draw call을 준비한다. GPU는 전달받은 명령을 바탕으로 vertex processing, primitive assembly, rasterization, fragment processing, framebuffer 쓰기를 순서대로 수행한다.

vertex shader가 각 vertex의 clip-space 위치와 varying을 계산하면 primitive가 조립·rasterize되고 fragment shader가 pixel 후보의 color/depth를 계산한다. CPU draw-call 제출과 GPU 실행은 비동기로 진행된다.

## 예시

```js
gl.useProgram(program);
gl.bindVertexArray(vao);
gl.bindTexture(gl.TEXTURE_2D, texture);
gl.drawElements(gl.TRIANGLES, indexCount, gl.UNSIGNED_SHORT, 0);
```

shader program·vertex state·texture를 binding한 뒤 indexed triangle draw call을 GPU command stream에 제출한다.

## 면접 답변 예시

> WebGL pipeline은 JavaScript가 state와 draw command를 제출하고 GPU가 vertex 처리, primitive 조립, rasterization과 fragment 처리를 거쳐 framebuffer에 쓰는 흐름입니다. 병목을 찾을 때 CPU의 draw-call 제출 비용과 GPU의 vertex·fragment·texture bandwidth를 frame capture로 나눠 보겠습니다. 작은 draw를 batching·instancing하고 불필요한 state change와 overdraw를 줄이되 `readPixels`처럼 CPU가 GPU 결과를 기다리는 동기화 지점은 피합니다. Buffer·texture 생명주기와 context lost·restore 경로를 테스트하고 해상도별 frame time과 GPU memory를 함께 측정합니다.

## 장점

- batching·instancing으로 draw call 수를 줄일 근거를 만든다.

## 단점

- overdraw와 복잡한 fragment shader는 고해상도에서 성능을 급격히 떨어뜨린다.

## 주의사항 / 실무 팁

- state change와 draw call을 material·texture별로 batch한다.
