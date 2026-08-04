# WebAuthn 정리

## 핵심 요약

- 공유 비밀번호 대신 사이트에 묶인 공개키 서명을 사용할 수 있다.
- challenge를 재사용하면 assertion replay를 막기 어렵다.
- challenge는 세션에 묶어 짧게 만료시키고 사용 즉시 폐기한다.

## 개념 설명

WebAuthn은 relying party가 authenticator의 공개키 credential로 등록과 로그인을 수행하게 하는 피싱 저항성 웹 인증 API다.

서버가 일회성 challenge를 만들고 `navigator.credentials`가 반환한 PublicKeyCredential의 attestation 또는 assertion을 서버가 RP ID, origin, 서명과 함께 검증한다.

## 예시

```ts
const credential = await navigator.credentials.get({
  publicKey: {
    challenge,
    rpId: "example.com",
    allowCredentials,
    userVerification: "required",
  },
});
if (!(credential instanceof PublicKeyCredential)) throw new Error("WebAuthn failed");
await verifyAssertionOnServer(credential);
```

브라우저에서 PublicKeyCredential 타입을 확인한 뒤 assertion 전체를 서버로 보낸다. challenge 재사용 방지와 서명 검증은 클라이언트가 아니라 서버 책임이다.

## 면접 답변 예시

> 서버 유출 시에도 authenticator의 개인키는 노출되지 않는다. 사용자 검증과 authenticator 정책이 과하면 등록 가능한 기기가 줄어든다. 등록 전후와 인증 실패를 보안 이벤트로 남기되 생체 정보는 수집하지 않는다.

## 장점

- 플랫폼 생체 인증과 외부 보안 키를 같은 API로 지원한다.

## 단점

- RP ID와 origin 검증을 생략하면 다른 사이트의 응답을 잘못 수용할 수 있다.

## 주의사항 / 실무 팁

- 서버에서 type, origin, rpIdHash, signature counter 정책을 검증한다.
