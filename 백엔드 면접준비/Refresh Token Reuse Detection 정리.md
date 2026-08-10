# Refresh Token Reuse Detection 정리

## 핵심 요약

- 탈취 refresh token을 빠르게 무효화할 수 있다.
- 정상 race를 공격으로 오탐하면 로그인 경험이 나빠진다.
- grace window와 기기 식별 기준을 함께 조정한다.

## 개념 설명

Refresh Token Reuse Detection은 이미 rotation된 refresh token이 다시 쓰였는지 찾아 탈취 가능성을 판단하는 보안 기능이다.

서버는 token family의 현재 version과 사용된 version을 비교하고, 만료·grace window 밖의 이전 version이면 family 전체를 폐기한다.

## 예시

```text
used_version < current_version and outside grace
-> mark token_family compromised
-> revoke active refresh tokens
```

이전 token 재사용은 단순 실패가 아니라 탈취 신호일 수 있어 family 단위 대응이 필요하다.

## 면접 답변 예시

> Refresh Token Reuse Detection은 rotation으로 이미 소비된 토큰이 다시 사용됐는지 확인하는 기능입니다. 정상적인 동시 요청은 짧은 grace window와 동일 client 조건으로 구분하고, 그 밖의 이전 generation 사용은 탈취 가능성이 있는 사건으로 봅니다. 의심되면 token family를 폐기하고 access token을 어떻게 차단할지도 함께 결정합니다. 저장소와 감사 로그에는 token 원문 대신 hash와 family·generation 정보만 남깁니다.

## 장점

- 사용자에게 의심 로그인 알림을 보낼 근거가 생긴다.

## 단점

- family 폐기가 access token 즉시 차단과 연결되지 않으면 창이 남는다.

## 주의사항 / 실무 팁

- reuse 탐지 후 access token revocation 전략을 정한다.
