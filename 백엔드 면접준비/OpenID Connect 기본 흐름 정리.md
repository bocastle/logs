# OpenID Connect 기본 흐름 정리

## 핵심 요약

- 로그인 정보를 표준 claim으로 받을 수 있다.
- id_token을 API 권한 token처럼 쓰면 권한 경계가 흐려진다.
- authorization code flow와 PKCE를 함께 사용한다.

## 개념 설명

OpenID Connect는 OAuth 2.0 위에 로그인 identity 정보를 표준화한 인증 계층이다.

클라이언트는 authorization code flow로 token을 받고, `id_token`의 issuer, audience, nonce, exp, 서명을 검증해 사용자를 인증한다.

## 예시

```text
authorize scope=openid profile
token response: id_token + access_token
validate iss, aud, exp, nonce, signature
```

`id_token`은 로그인 증명이고, API 호출 권한 판단은 access token과 scope로 분리해야 한다.

## 면접 답변 예시

> OpenID Connect는 OAuth 2.0 authorization code flow 위에서 사용자의 login identity를 ID token claim으로 전달하는 인증 계층입니다. Client는 PKCE와 state를 사용하고 ID token의 signature, issuer, audience, expiration과 요청에 묶은 nonce를 모두 검증하겠습니다. ID token은 로그인 증명이지 API authorization token이 아니므로 API에는 access token과 scope를 사용합니다. Email과 profile claim은 바뀔 수 있어 issuer와 stable subject를 내부 account에 매핑하고 JWKS key rotation도 처리합니다.

## 장점

- nonce 검증으로 로그인 응답 재사용을 줄인다.

## 단점

- nonce 검증을 빼면 replay 위험이 커진다.

## 주의사항 / 실무 팁

- id_token 검증 실패는 상세 원인을 외부에 노출하지 않는다.
