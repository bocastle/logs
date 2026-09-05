# Service Worker Update UX 정리

## 핵심 요약

- 브라우저 내부 생명주기를 사용자가 이해할 수 있는 단계로 보여줄 수 있다.
- 첫 설치와 업데이트를 구분하지 않으면 새 방문에도 갱신 배너가 나타난다.
- 등록 직후 registration.waiting과 installing을 모두 초기 점검한다.

## 개념 설명

Service Worker Update UX는 새 worker의 발견, 설치, 대기, 제어권 전환을 사용자 화면의 준비·적용·완료 상태로 번역하는 흐름이다.

registration의 `updatefound`에서 installing worker를 관찰하고 installed 뒤 기존 controller가 있으면 `waiting` 상태를 알리며, `controllerchange`에서 적용 완료를 처리한다.

## 예시

```ts
registration.addEventListener("updatefound", () => {
  const installing = registration.installing;
  installing?.addEventListener("statechange", () => {
    if (installing.state === "installed" && navigator.serviceWorker.controller) {
      showReadyToUpdate(registration.waiting);
    }
  });
});
navigator.serviceWorker.addEventListener("controllerchange", finishUpdateOnce);
```

updatefound만으로 완료를 선언하지 않고 installing.state를 확인한다. 기존 controller가 있는 installed 상태가 실제 업데이트 대기 시점이다.

## 면접 답변 예시

> Service Worker update UX는 새 worker의 installing, waiting과 controlling 상태를 사용자가 이해할 준비·적용·완료 단계로 바꾸는 일입니다. `updatefound`만으로 완료라고 하지 않고 installed 상태와 기존 controller를 확인해 첫 설치와 update를 구분하겠습니다. Waiting worker를 적용한 뒤에는 `controllerchange`를 실제 제어권 전환 신호로 사용합니다. 이 event마다 무조건 reload하면 반복 새로고침이 생길 수 있어 session당 한 번만 처리하고, 사용자가 작성 중인 data가 있다면 적용 시점도 선택하게 합니다.

## 장점

- 설치 실패와 적용 완료를 분리해 안내할 수 있다.

## 단점

- statechange 리스너를 늦게 붙이면 이미 진행된 상태를 놓칠 수 있다.

## 주의사항 / 실무 팁

- 설치 오류는 개발자 로그와 사용자 재시도 안내를 나눠 기록한다.
