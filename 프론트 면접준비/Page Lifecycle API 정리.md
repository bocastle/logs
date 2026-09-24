# Page Lifecycle API 정리

## 핵심 요약

- 백그라운드 page의 CPU·battery·lock 점유를 줄인다.
- lifecycle event 순서를 하나로 가정하면 mobile 종료에서 저장을 놓친다.
- 중요한 draft는 상태 변경 시점보다 작업 중 점진적으로 저장한다.

## 개념 설명

Page Lifecycle API는 page가 active·passive·hidden·frozen·discarded 상태로 이동할 때 작업과 리소스를 정리·재개하는 lifecycle 모델과 이벤트 모음이다.

`visibilitychange`, `pagehide`, `pageshow`를 기본으로 쓰고 지원 환경에서 `freeze`/`resume`에 맞춰 timer·IndexedDB transaction·Web Lock·connection을 정리한다. `unload`에 필수 로직을 두지 않는다.

## 예시

```js
document.addEventListener("freeze", releaseEphemeralResources);
document.addEventListener("resume", revalidateVisibleState);
window.addEventListener("pagehide", persistSmallDraft);
```

freeze 전에 임시 리소스를 풀고 resume 후 사용자에게 보이는 상태를 재검증한다.

## 면접 답변 예시

> 상태별 정리·저장·재검증 책임을 명시한다. `unload` listener는 bfcache 적격성을 해칠 수 있다. background·freeze·bfcache·discard 후 포그라운드 복귀를 테스트한다.

## 장점

- bfcache와 freeze 복귀를 새 page load와 구분한다.

## 단점

- freeze에서 시간이 긴 로직을 실행하면 완료되지 못할 수 있다.

## 주의사항 / 실무 팁

- release·resume handler를 짧고 idempotent하게 만든다.
