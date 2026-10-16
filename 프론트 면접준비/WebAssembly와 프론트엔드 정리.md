# WebAssembly와 프론트엔드 정리

## 핵심 요약

- WebAssembly는 계산량이 큰 코드를 브라우저의 저수준 실행 환경에서 돌리기 위한 바이너리 형식이다.
- DOM을 직접 다루는 대체재라기보다 이미지 처리, 압축, 에뮬레이션처럼 계산 경계가 분명한 작업에 잘 맞는다.
- 성능은 연산 속도뿐 아니라 JavaScript 경계를 넘는 호출 수와 메모리 복사 비용까지 함께 측정해야 한다.

## 개념 설명

WebAssembly 모듈은 샌드박스 안에서 실행되고, 숫자 중심의 함수와 선형 메모리를 통해 JavaScript와 데이터를 주고받는다.

작은 함수를 자주 호출하면 경계 비용이 커질 수 있으므로 데이터를 묶어서 넘기고, 메모리 소유권과 해제 시점을 명확히 하는 편이 좋다.

## 예시

```ts
const { instance } = await WebAssembly.instantiateStreaming(
  fetch("/image-filter.wasm"),
  imports,
);
const memory = instance.exports.memory as WebAssembly.Memory;
const pixels = new Uint8Array(memory.buffer, inputPtr, inputLength);
pixels.set(sourcePixels);
instance.exports.applyFilter(inputPtr, inputLength);
```

픽셀을 한 번에 선형 메모리로 복사한 뒤 연산을 호출하는 예다. 호출을 픽셀마다 반복하면 JavaScript 경계 비용이 이점을 삼킬 수 있다.

## 면접 답변 예시

> WebAssembly는 browser UI 전체를 대체하기보다 image 처리나 압축처럼 계산 경계가 분명한 부분을 분리하는 선택지입니다. DOM 접근은 JavaScript를 거쳐야 하고 작은 함수를 자주 호출하면 경계 비용이 연산 이점을 삼킬 수 있습니다. 먼저 profile로 실제 병목을 확인한 뒤 module 크기와 초기 compile 시간, 선형 memory 복사 비용까지 JavaScript 구현과 비교하겠습니다. 지원이나 loading이 실패할 때 사용할 fallback도 함께 준비합니다.

## 장점

- C·C++·Rust로 작성된 계산 코드를 웹에서 재사용할 수 있다.
- 연산 경계가 큰 작업에서는 예측 가능한 성능을 얻기 좋다.
- 브라우저 샌드박스 안에서 네이티브에 가까운 계산 모델을 사용할 수 있다.

## 단점

- 모듈 다운로드와 컴파일이 초기 로딩 비용을 늘린다.
- DOM 접근은 JavaScript를 거쳐야 해서 UI 작업에는 이점이 작다.
- 선형 메모리와 디버깅 도구에 익숙하지 않으면 유지보수 비용이 커진다.

## 주의사항 / 실무 팁

- 도입 전에 실제 사용자 기기에서 JavaScript 구현과 벤치마크한다.
- 작은 호출을 반복하지 말고 입력을 묶어 경계를 넘긴다.
- 모듈 로딩 실패 시 사용할 JavaScript fallback이나 기능 제한 흐름을 정한다.
