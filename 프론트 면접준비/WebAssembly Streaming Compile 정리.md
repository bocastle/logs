# WebAssembly Streaming Compile 정리

## 핵심 요약

- wasm 모듈 초기화 대기 시간을 줄일 수 있다.
- 서버 MIME type이 틀리면 streaming이 실패할 수 있다.
- `Content-Type: application/wasm`을 확인한다.

## 개념 설명

WebAssembly Streaming Compile은 wasm 바이트를 모두 받은 뒤가 아니라 다운로드와 컴파일을 겹쳐 초기화 시간을 줄이는 방식이다.

`WebAssembly.instantiateStreaming(fetch(url), imports)`는 서버가 올바른 `application/wasm` MIME type을 줄 때 효과가 가장 좋다.

## 예시

```js
const { instance } = await WebAssembly.instantiateStreaming(fetch("/image.wasm"), imports);
```

이미지 처리 모듈처럼 초기화가 무거운 wasm을 다운로드와 동시에 컴파일한다. MIME type이 틀리면 fallback 경로가 필요하다.

## 면접 답변 예시

> WebAssembly streaming compile은 wasm 전체 다운로드가 끝날 때까지 기다리지 않고 network 수신과 compile을 겹쳐 초기화 시간을 줄이는 방식입니다. `WebAssembly.instantiateStreaming()`을 사용하려면 정상 HTTP 응답과 `Content-Type: application/wasm`을 확인하고 imports 계약 오류도 별도로 처리하겠습니다. Streaming을 사용할 수 없거나 MIME이 잘못된 환경에는 response의 `ArrayBuffer`를 받아 `WebAssembly.instantiate()`하는 fallback을 준비합니다. 모듈 크기, fetch·compile·instantiate 시간을 RUM과 lab에서 나눠 측정해 작은 module에도 복잡성을 추가할 가치가 있는지 판단합니다.

## 장점

- 다운로드와 컴파일을 겹쳐 CPU와 네트워크 시간을 활용한다.

## 단점

- 작은 모듈은 개선보다 복잡도가 더 클 수 있다.

## 주의사항 / 실무 팁

- instantiate fallback을 준비한다.
