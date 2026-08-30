# Credential Management API 정리

## 핵심 요약

- 브라우저와 비밀번호 관리자의 로그인 선택 UI를 활용할 수 있다.
- null 결과를 로그인 실패로 단정하면 사용자가 직접 입력할 기회를 잃는다.
- 반환값이 null인 경우 일반 로그인 폼을 그대로 제공한다.

## 개념 설명

Credential Management API는 브라우저의 credential 저장소와 웹 로그인 화면 사이에서 비밀번호, 연합 ID, OTP, 공개키 credential을 요청하거나 저장하는 공통 인터페이스다.

`navigator.credentials.get`의 `mediation` 옵션은 자동·선택적 UI 동작을 조절하고 PasswordCredential 같은 결과는 타입을 확인한 뒤 해당 인증 서버 흐름으로 전달해야 한다.

## 예시

```ts
const credential = await navigator.credentials.get({
  password: true,
  mediation: "optional",
});
if (credential instanceof PasswordCredential) {
  await signInWithPassword(credential.id, credential.password);
}
```

optional mediation으로 저장된 비밀번호 credential을 요청하고 PasswordCredential인 경우에만 비밀번호 로그인 흐름을 실행한다.

## 면접 답변 예시

> Credential Management API는 browser와 password manager가 가진 credential 선택을 web login 흐름에 연결하는 인터페이스입니다. `mediation`으로 silent access와 사용자 선택의 경계를 정하되, `get()`이 `null`을 반환한 것을 인증 실패로 단정하지 않고 일반 login form을 그대로 제공하겠습니다. 반환된 credential은 타입별로 분기해 각 server 검증 흐름으로 보내고 password나 token을 client log에 남기지 않습니다. Logout 시에는 `preventSilentAccess()`를 검토해 사용자가 명시적으로 나간 직후 자동 로그인이 반복되지 않게 합니다.

## 장점

- 서로 다른 credential 유형을 하나의 CredentialsContainer에서 요청할 수 있다.

## 단점

- credential 타입을 확인하지 않으면 다른 인증 방식의 결과를 잘못 처리한다.

## 주의사항 / 실무 팁

- PasswordCredential과 PublicKeyCredential을 분기해 각 서버 검증으로 보낸다.
