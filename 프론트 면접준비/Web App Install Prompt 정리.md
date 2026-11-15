# Web App Install Prompt 정리

## 핵심 요약

- 사용자가 앱 가치를 경험한 화면에서 설치를 제안할 수 있다.
- 이벤트가 오지 않는 환경에서 빈 설치 버튼을 노출할 수 있다.
- beforeinstallprompt 수신 전에는 자체 설치 버튼을 숨긴다.

## 개념 설명

Web App Install Prompt는 브라우저가 설치 가능하다고 판단해 보낸 beforeinstallprompt 이벤트를 앱이 보관하고 가치가 분명한 사용자 동작에서 설치 선택을 요청하는 흐름이다.

이벤트의 기본 표시를 막고 한 번 저장한 뒤 설치 버튼에서 `prompt()`를 호출하며 `userChoice` 결과를 기록한 다음 참조를 비운다.

## 예시

```ts
let installEvent: BeforeInstallPromptEvent | null = null;
addEventListener("beforeinstallprompt", (event) => {
  event.preventDefault();
  installEvent = event;
  installButton.hidden = false;
});
installButton.addEventListener("click", async () => {
  await installEvent?.prompt();
  const choice = await installEvent?.userChoice;
  recordInstallChoice(choice?.outcome);
  installEvent = null;
});
```

beforeinstallprompt를 보관했다가 설치 버튼에서 prompt()와 userChoice를 순서대로 사용한다. 이벤트는 재사용할 수 없으므로 처리 뒤 null로 만든다.

## 면접 답변 예시

> Web App Install Prompt는 브라우저가 설치 가능하다고 알린 경우 그 event를 저장했다가 사용자가 앱의 가치를 경험한 시점에 자체 설치 버튼으로 요청하는 흐름입니다. `beforeinstallprompt`가 오기 전에는 버튼을 숨기고, 지원하지 않거나 설치 조건을 충족하지 못한 환경에는 빈 동작을 노출하지 않겠습니다. 사용자 click에서 `prompt()`와 `userChoice`를 한 번 처리한 뒤 저장한 event를 비워 반복 호출을 막습니다. `appinstalled`에서는 배너와 상태를 정리하고 설치 제어가 없는 브라우저에는 기본 설치 안내나 일반 웹 사용 흐름을 유지합니다.

## 장점

- 설치 수락과 거절 결과를 제품 지표로 구분할 수 있다.

## 단점

- 페이지 진입 직후 prompt를 호출하면 거절률과 피로가 커진다.

## 주의사항 / 실무 팁

- 저장, 오프라인 사용 등 설치 이점이 드러난 뒤 버튼을 제시한다.
