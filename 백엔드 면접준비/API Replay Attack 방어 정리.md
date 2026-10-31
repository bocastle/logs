# API Replay Attack 방어 정리

## 핵심 요약

- 결제·권한 변경 같은 민감 API의 중복 실행을 막는다.
- nonce 저장소 TTL이 짧으면 지연된 replay를 놓친다.
- 허용 clock skew를 문서화하고 서버 시간을 응답에 제공한다.

## 개념 설명

API Replay Attack 방어는 유효했던 요청을 공격자가 다시 보내 같은 효과를 반복하지 못하게 막는 설계다.

timestamp 허용 범위, nonce 저장소, idempotency key, request signing을 조합해 같은 요청의 재사용을 탐지한다.

## 예시

```text
reject if timestamp older than 5m
reject if nonce already used for client_id
verify signature over nonce and body hash
```

`nonce`가 서명 대상에 들어가야 공격자가 값을 바꿔 replay 탐지를 우회할 수 없다.

## 면접 답변 예시

> Replay attack은 한 번 유효했던 민감 request를 공격자가 다시 보내 결제나 권한 변경을 반복하는 공격입니다. Client ID, timestamp, nonce와 body hash를 같은 signature에 포함하고 server는 clock window와 nonce 미사용 여부를 함께 검증하겠습니다. Nonce TTL은 허용 timestamp window보다 짧지 않게 두고 request idempotency key는 정상 retry의 중복 부작용도 막는 별도 역할로 사용합니다. Nonce store 장애 시 민감 API는 보수적으로 차단하고 거절 event는 secret 없이 보안 alert와 audit log에 연결합니다.

## 장점

- 탈취된 패킷의 재사용 시간을 짧게 제한한다.

## 단점

- 시간 동기화가 나쁘면 정상 요청도 거부된다.

## 주의사항 / 실무 팁

- nonce 저장소 장애 시 민감 API는 보수적으로 차단한다.
