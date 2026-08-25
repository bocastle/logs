# OAuth PKCE 정리

## 핵심 요약

- 모바일·SPA 같은 public client의 code 탈취 위험을 줄인다.
- plain challenge를 허용하면 보호 효과가 낮다.
- 가능하면 `S256`만 허용한다.

## 개념 설명

OAuth PKCE는 authorization code를 가로챈 공격자가 token으로 교환하지 못하게 하는 OAuth 확장이다.

클라이언트는 `code_verifier`에서 `code_challenge`를 만들고, token 교환 시 원래 verifier를 보내 authorization server가 검증한다.

## 예시

```text
code_challenge = BASE64URL(SHA256(code_verifier))
authorize: code_challenge_method=S256
token: code_verifier=<original random>
```

`S256` 방식은 verifier 원문을 authorization 요청에 노출하지 않아 public client에 적합하다.

## 면접 답변 예시

> PKCE는 authorization code를 가로챈 공격자가 그 code만으로 token을 교환하지 못하게 하는 장치입니다. Client가 무작위 `code_verifier`에서 `S256` challenge를 만들어 authorization 요청에 보내고, token 요청에서 원래 verifier를 증명합니다. 특히 secret을 안전하게 보관하기 어려운 SPA와 mobile client에 필수적이지만, PKCE가 redirect 공격이나 로그인 CSRF까지 모두 해결하는 것은 아닙니다. Exact redirect URI 검증과 state를 함께 적용하고 authorization code는 짧게 만료되는 일회용으로 처리하겠습니다.

## 장점

- client secret 없이도 token 교환을 보호할 수 있다.

## 단점

- verifier 길이와 난수 품질이 약하면 추측 위험이 커진다.

## 주의사항 / 실무 팁

- state 검증과 redirect URI 엄격 매칭을 함께 적용한다.
