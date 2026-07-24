# SharedArrayBuffer 사용 전 보안 조건 체크 정리

## 핵심 요약

- `SharedArrayBuffer`는 메인 스레드와 worker가 같은 메모리를 공유하지만 일반적인 데이터 경쟁 위험도 함께 가져온다.
- 브라우저에서는 COOP·COEP를 포함한 cross-origin isolation을 먼저 구성하고 `crossOriginIsolated`를 확인한다.
- 헤더 변경이 popup, iframe과 서드파티 리소스에 미치는 영향을 점검하고 복사 기반 fallback을 남긴다.

## 개념 설명

`SharedArrayBuffer`는 여러 실행 컨텍스트가 같은 바이트를 복사 없이 읽고 쓰게 한다. WebAssembly thread나 대용량 계산에는 유용하지만 잘못된 동기화로 data race와 교착 상태를 만들 수 있다.

브라우저는 관련 보안 위험을 줄이기 위해 cross-origin isolated 문맥을 요구한다. 보통 COOP와 COEP header를 설정하고 모든 cross-origin 하위 리소스가 정책을 만족하게 해야 한다. Header만 넣었다고 가정하지 말고 runtime의 `crossOriginIsolated`로 최종 상태를 확인한다.

## 예시

```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

```ts
if (crossOriginIsolated) {
  const shared = new SharedArrayBuffer(1024);
}
```

COEP를 켜면 적절한 CORS 또는 CORP 정책이 없는 외부 script와 image가 차단될 수 있다. Staging에서 실제 resource 목록과 OAuth popup 같은 opener 관계를 확인한 뒤 기능을 켠다.

## 면접 답변 예시

> `SharedArrayBuffer`는 메인 스레드와 worker가 복사 없이 같은 메모리를 공유하는 기능입니다. 브라우저에서는 COOP와 COEP로 cross-origin isolation을 구성하고 `crossOriginIsolated`가 true인지 확인한 뒤 사용하겠습니다. 이 header는 서드파티 리소스와 popup 관계를 바꿀 수 있어 staging에서 전체 resource를 점검해야 합니다. 공유 메모리 접근은 `Atomics`와 명확한 소유권·종료 protocol로 제한하고 미지원 환경에는 `ArrayBuffer` 전송 경로를 남깁니다.

## 장점

- Worker 사이의 대용량 복사를 줄이고 `Atomics` 기반 병렬 계산을 구성할 수 있다.
- WebAssembly thread 같은 고성능 작업의 기반이 된다.

## 단점

- Data race, 교착과 종료 순서 같은 메모리 동기화 버그는 재현이 어렵다.
- COOP·COEP 적용으로 기존 popup과 cross-origin embed가 동작하지 않을 수 있다.

## 주의사항 / 실무 팁

- `crossOriginIsolated`가 false인 환경에는 transferable `ArrayBuffer` 같은 fallback을 남긴다.
- 공유 영역의 소유권, `Atomics` 사용 위치와 worker 종료 신호를 protocol로 문서화한다.
