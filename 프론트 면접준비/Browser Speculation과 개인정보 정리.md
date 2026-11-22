# Browser Speculation과 개인정보 정리

## 핵심 요약

- 예측 가능한 다음 경로를 빠르게 열 수 있다.
- prerender 중 개인정보성 요청을 보내면 의도보다 이르게 노출될 수 있다.
- prerender 상태에서는 analytics와 민감 요청을 지연한다.

## 개념 설명

Browser speculation은 prerender나 prefetch 후보를 브라우저에 알려 다음 탐색을 앞당기되 개인정보성 요청 시점을 관리해야 하는 플랫폼 동작이다.

Speculation Rules로 후보 URL을 선언하면 브라우저가 보이지 않는 문서를 미리 준비할 수 있으므로 analytics, 인증 갱신, 민감 요청은 실제 활성화 뒤로 늦춘다.

`pageshow`는 prerender 단계에서도 발생할 수 있어 사용자가 실제로 문서를 봤다는 신호로 쓰기 어렵다. 활성화 시점은 `prerenderingchange`로 감지하고, 이미 활성화된 뒤 실행된 코드에서는 `PerformanceNavigationTiming.activationStart`가 0보다 큰지 확인한다.

## 예시

```ts
async function afterActivation() {
  if (document.prerendering) {
    await new Promise<void>((resolve) =>
      document.addEventListener("prerenderingchange", () => resolve(), { once: true }),
    );
  }
  startAnalytics();
  refreshSensitiveData();
}
```

prerender 중이면 활성화 event를 기다렸다가 analytics와 사용자 의도가 필요한 민감 요청을 시작한다.

## 면접 답변 예시

> Browser speculation은 다음 문서를 미리 fetch하거나 prerender해 탐색을 빠르게 하지만 사용자가 방문하기 전에 URL과 request가 server에 도달할 수 있습니다. 계정 설정이나 일회성 action처럼 민감한 경로는 후보에서 제외하고 prerender 상태에서는 analytics, 인증 갱신과 외부 side effect를 시작하지 않겠습니다. `document.prerendering`이 true이면 `prerenderingchange`를 기다리고, 이미 활성화됐다면 `activationStart`를 확인합니다. `pageshow`는 prerender 중에도 발생할 수 있으므로 활성화 신호로 사용하지 않습니다. DevTools에서 후보·실패 이유를 확인하고 prerender hit rate, 취소 비용과 활성화 전 발생한 network 요청을 함께 검수합니다.

## 장점

- 실제 탐색이 적중하면 문서와 하위 리소스가 준비된 상태에서 거의 즉시 화면을 보여 줄 수 있다.

## 단점

- 활성화되지 않은 후보도 네트워크와 메모리를 사용하며 URL 요청 자체가 사용자 의도를 미리 노출할 수 있다.

## 주의사항 / 실무 팁

- 결제·삭제·로그아웃처럼 부작용이 있는 URL과 개인정보성 경로는 후보에서 제외한다.
- `document.prerendering`, `prerenderingchange`, `activationStart`를 조합해 활성화 전후 요청을 점검한다.
