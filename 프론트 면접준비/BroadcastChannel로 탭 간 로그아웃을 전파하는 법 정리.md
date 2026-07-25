# BroadcastChannel로 탭 간 로그아웃을 전파하는 법 정리

## 핵심 요약

- 열린 여러 탭의 로그아웃 상태를 짧은 지연으로 맞출 수 있다.
- 나중에 열린 탭은 과거 logout 메시지를 받지 못한다.
- 앱 시작 때 서버 세션 상태를 다시 확인한다.

## 개념 설명

BroadcastChannel 로그아웃 전파는 한 탭에서 세션을 끝낸 사실을 같은 origin의 열린 탭에 알려 인증 UI와 로컬 상태를 즉시 정리하는 패턴이다.

로그아웃 요청이 서버에서 완료되면 `postMessage`로 `logout` 이벤트와 발생 시각만 보내고, 수신 탭은 캐시를 비운 뒤 로그인 화면으로 이동한다.

## 예시

```ts
const authChannel = new BroadcastChannel("bocelog:auth:v1");
async function logout() {
  await revokeSession();
  authChannel.postMessage({ type: "logout", at: Date.now() });
  clearSessionAndRedirect();
}
authChannel.onmessage = ({ data }) => {
  if (data.type === "logout") clearSessionAndRedirect();
};
```

서버 세션을 먼저 폐기한 뒤 BroadcastChannel로 logout 사실만 알린다. 보낸 탭은 자기 메시지를 받지 않으므로 로컬 정리 함수를 직접 호출한다.

## 면접 답변 예시

> 수신 탭마다 인증 화면 갱신 로직을 재사용할 수 있다. 수신 처리 중 저장되지 않은 입력을 예고 없이 잃을 수 있다. 편집 화면에서는 로컬 초안을 보존한 뒤 인증 만료 안내로 이동한다.

## 장점

- 토큰 저장소 변경 감시보다 메시지 의도가 명확하다.

## 단점

- 서버 폐기 전에 메시지를 보내면 다른 탭만 로그아웃되고 요청 탭의 세션이 남을 수 있다.

## 주의사항 / 실무 팁

- 로그아웃 정리 함수가 여러 번 호출되어도 안전하게 만든다.
