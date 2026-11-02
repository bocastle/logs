# OAuth Token Introspection 정리

## 핵심 요약

- opaque token도 중앙에서 상태를 확인할 수 있다.
- 매 요청 introspection은 인증 서버 병목을 만든다.
- 고위험 scope는 cache TTL을 짧게 둔다.

## 개념 설명

OAuth Token Introspection은 resource server가 authorization server에 token 상태와 scope를 물어보는 표준 흐름이다.

resource server는 introspection endpoint에 token을 보내 `active`, subject, scope, exp를 확인하고 결과를 짧게 cache할 수 있다.

## 예시

```json
{"active":true,"sub":"user_7","scope":"orders:read","exp":1780000000}
```

`active`가 false이면 서명 형식이 맞아 보여도 resource server는 요청을 거부해야 한다.

## 면접 답변 예시

> OAuth token introspection은 resource server가 authorization server에 opaque token의 `active`, subject, scope와 expiration을 확인하는 표준 흐름입니다. Introspection endpoint 자체는 인증된 resource server만 호출할 수 있게 하고 token과 응답을 log에 노출하지 않겠습니다. 매 request 원격 조회는 auth server 병목이 되므로 risk와 scope에 맞는 짧은 cache를 쓰되 그 TTL만큼 revocation 반영이 늦어지는 tradeoff를 명시합니다. Endpoint 장애 때 고위험 API는 보수적으로 거절하고 latency와 error rate를 인증 SLO에 포함합니다.

## 장점

- token revocation을 비교적 빠르게 반영할 수 있다.

## 단점

- cache TTL이 길면 폐기 반영이 늦어진다.

## 주의사항 / 실무 팁

- resource server 인증과 mTLS 적용을 검토한다.
