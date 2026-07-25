# Service Worker 캐시 업데이트 알림을 설계하는 법 정리

## 핵심 요약

- 사용자가 작성 중인 내용을 저장한 뒤 갱신할 수 있다.
- 대기 worker가 바뀐 뒤 예전 참조에 메시지를 보내면 적용되지 않을 수 있다.
- 적용 직전에 폼과 편집기 초안을 저장한다.

## 개념 설명

Service Worker 캐시 업데이트 알림은 새 worker가 설치되어 대기 중일 때 사용자의 현재 작업을 보존하면서 새 자산 적용 시점을 선택하게 하는 UI다.

페이지는 `registration.waiting`을 확인해 갱신 배너를 열고 사용자가 적용을 누르면 waiting worker에 `postMessage`로 `skipWaiting` 명령을 보낸다.

## 예시

```ts
function showUpdate(registration: ServiceWorkerRegistration) {
  if (!registration.waiting) return;
  updateButton.onclick = () => {
    registration.waiting?.postMessage({ type: "SKIP_WAITING" });
  };
  updateBanner.hidden = false;
}
```

registration.waiting이 있을 때만 적용 버튼을 노출한다. worker는 명시된 메시지를 받은 경우에만 skipWaiting을 호출하도록 구성한다.

## 면접 답변 예시

> 강제 새로고침 대신 예측 가능한 배포 안내를 제공한다. 배너만 닫고 갱신을 영구 미루면 사용자가 오래된 코드를 계속 사용한다. 배너를 닫은 뒤에도 다음 안전한 화면 전환에서 다시 알린다.

## 장점

- 새 캐시가 준비된 상태와 실제 적용 상태를 구분할 수 있다.

## 단점

- 여러 탭이 서로 다른 시점에 갱신하면 자산 버전이 잠시 섞일 수 있다.

## 주의사항 / 실무 팁

- controllerchange를 한 번만 처리해 새로고침 루프를 막는다.
