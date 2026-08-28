# Token Revocation 정리

## 핵심 요약

- 로그아웃과 계정 침해 대응이 명확해진다.
- 모든 요청마다 중앙 denylist를 보면 지연이 커진다.
- access token 수명은 폐기 요구와 성능 사이에서 정한다.

## 개념 설명

Token Revocation은 발급된 token을 만료 전에도 더 이상 사용할 수 없게 만드는 절차다.

서버는 refresh token 저장소를 폐기하고, access token은 짧은 만료 시간, denylist, introspection 중 하나로 폐기 전파 지연을 줄인다.

## 예시

```text
logout -> revoke refresh token
high risk -> add jti to denylist until exp
resource server checks denylist for sensitive scope
```

`jti` denylist는 access token 만료 전 즉시 차단이 필요한 민감 scope에 선별 적용하는 편이 비용을 줄인다.

## 면접 답변 예시

> Token revocation은 발급된 token을 원래 만료 시각보다 먼저 사용할 수 없게 만드는 절차입니다. Logout에서는 refresh token을 폐기하고 access token은 짧게 유지하는 것이 기본이며, 즉시 차단이 필요한 고위험 scope에는 `jti` denylist나 introspection을 선택적으로 적용하겠습니다. 모든 요청이 중앙 저장소를 조회하면 latency와 가용성 의존성이 커지므로 위험도에 따라 비용을 나눠야 합니다. 여러 resource server에 revocation event를 전파한다면 지연과 실패를 관측하고 그 사이 허용되는 사용 창도 정책에 명시합니다.

## 장점

- 기기별 세션 종료를 구현할 수 있다.

## 단점

- access token exp가 길면 폐기 후에도 사용 창이 남는다.

## 주의사항 / 실무 팁

- 민감 API만 denylist 조회를 강하게 적용한다.
