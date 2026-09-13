# Session Fixation 방어 정리

## 핵심 요약

- 로그인 전후 세션 탈취 위험을 줄인다.
- 이전 session id를 남겨 두면 공격 창이 유지된다.
- 인증 성공 직후 session id를 원자적으로 교체한다.

## 개념 설명

Session Fixation 방어는 공격자가 미리 만든 session id를 사용자가 로그인 후에도 그대로 쓰게 하는 공격을 막는 설계다.

로그인, 권한 상승, MFA 완료 시 session id를 재발급하고 이전 id는 즉시 무효화한다.

## 예시

```text
before login: sid=anon_123
after login: rotate sid=user_789
invalidate anon_123
```

인증 경계에서 `sid`를 바꾸지 않으면 공격자가 알고 있던 익명 session이 로그인 session으로 승격될 수 있다.

## 면접 답변 예시

> Session fixation은 공격자가 알고 있는 익명 session ID를 피해자가 로그인한 뒤에도 그대로 쓰게 만드는 공격입니다. 인증 성공, MFA 완료와 권한 상승 같은 신뢰 경계에서 session ID를 새로 발급하고 이전 ID는 즉시 무효화하겠습니다. 필요한 비인증 상태만 안전하게 새 session으로 옮기고 rotation과 인증 상태 변경이 어긋나지 않게 framework의 session protection을 사용합니다. Cookie에는 `HttpOnly`, `Secure`, 적절한 `SameSite`를 적용하고 동시 요청 중 이전 session이 다시 살아나지 않는지도 테스트합니다.

## 장점

- 권한 상승 시 세션 경계를 새로 만들 수 있다.

## 단점

- 동시 요청 처리 중 rotation이 어긋나면 사용자가 로그아웃될 수 있다.

## 주의사항 / 실무 팁

- 쿠키에는 HttpOnly, Secure, SameSite를 적용한다.
