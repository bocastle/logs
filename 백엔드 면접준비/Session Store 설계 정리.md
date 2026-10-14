# Session Store 설계 정리

## 핵심 요약

- 서버에서 즉시 세션 폐기가 가능하다.
- 저장소 장애가 곧 로그인 장애가 될 수 있다.
- session id는 충분히 긴 난수로 만들고 hash 저장을 검토한다.

## 개념 설명

Session Store는 서버가 session id와 사용자 상태, 만료, 기기 정보를 저장하는 저장소다.

세션 값은 최소 권한 정보만 담고, sliding expiration, absolute expiration, 기기별 revocation을 분리해 관리한다.

## 예시

```text
sid -> {user_id, tenant_id, scopes, idle_exp, absolute_exp, device_id}
rotate sid on login and privilege change
```

`idle_exp`와 `absolute_exp`를 나누면 활동 중 세션 유지와 장기 탈취 위험 제한을 동시에 다룰 수 있다.

## 면접 답변 예시

> Server-side session store는 random session ID에 사용자와 tenant, 최소 권한 상태와 만료를 연결해 즉시 revoke할 수 있게 합니다. Login, MFA와 권한 상승 때 ID를 rotate하고 idle expiration과 absolute expiration을 분리하겠습니다. Cookie의 ID가 유출되면 그대로 bearer credential이 되므로 충분한 entropy와 `Secure`, `HttpOnly`, `SameSite`를 적용하고 저장소에는 ID hash 보관도 검토합니다. Store latency와 오류가 인증 경로 장애가 되므로 SLO와 revocation·policy 변경 전파 시간을 함께 관리합니다.

## 장점

- 기기별 로그인 목록과 강제 로그아웃을 만들 수 있다.

## 단점

- 세션에 많은 상태를 넣으면 동기화 비용이 커진다.

## 주의사항 / 실무 팁

- 세션 저장소 latency와 error rate를 인증 SLO에 포함한다.
