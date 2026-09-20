# Device Memory API 정리

## 핵심 요약

- 저사양 기기에서 과한 병렬 작업을 줄일 수 있다.
- bucket 값은 정밀하지 않아 성능 등급을 완전히 보장하지 않는다.
- navigator.deviceMemory가 없을 때의 기본 worker 수를 정한다.

## 개념 설명

Device Memory API는 브라우저가 기기 RAM을 거친 bucket 값으로 노출해 기능 강도를 보수적으로 조절하게 하는 클라이언트 힌트 API다.

`navigator.deviceMemory`는 정확한 메모리 측정값이 아니라 개인정보 보호를 위해 반올림된 숫자이므로 worker 수, 미디어 품질, 캐시 한도를 낮추는 힌트로만 써야 한다.

## 예시

```ts
const memoryBucket = navigator.deviceMemory ?? 4;
const workerCount = memoryBucket <= 2 ? 1 : 2;
const previewPixels = memoryBucket <= 2 ? 1_000_000 : 4_000_000;
```

낮은 deviceMemory bucket에서는 worker 수와 미리보기 픽셀 한도를 줄인다. 값이 없으면 중간 사양 기본값으로 처리해 기능을 막지 않는다.

## 면접 답변 예시

> 기능 강도 결정을 사용자 기기 조건과 연결해 설명할 수 있다. 메모리 압박 자체의 원인 분석은 이미지, heap, GPU 리소스 지표와 별도로 봐야 한다. Browser Memory Pressure 대응은 캐시 해제와 리소스 정리 규칙으로 분리한다.

## 장점

- 정확한 RAM 수집 없이 브라우저가 허용한 힌트만 사용한다.

## 단점

- 값이 없는 브라우저도 많아 기본 경로가 필요하다.

## 주의사항 / 실무 팁

- bucket별 품질 제한이 과하게 낮지 않은지 실제 기기에서 확인한다.
