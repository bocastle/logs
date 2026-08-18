# mTLS 정리

## 핵심 요약

- 서비스 간 호출에서 네트워크 위조를 줄인다.
- 인증서 회전 실패가 곧 통신 장애가 된다.
- 인증서 만료 알림과 자동 회전 테스트를 둔다.

## 개념 설명

mTLS는 클라이언트와 서버가 서로 인증서를 검증해 양방향 신뢰를 만드는 TLS 설정이다.

서버는 client certificate chain, SAN, 만료, revocation 상태를 확인하고 인증서 identity를 서비스나 client id로 매핑한다.

## 예시

```text
client cert SAN=spiffe://prod/payment-api
server verifies CA chain and maps SAN to service account
```

SAN을 정책 입력으로 써야 인증서가 유효하다는 사실과 어떤 서비스인지 판단하는 일을 연결할 수 있다.

## 면접 답변 예시

> mTLS는 server와 client가 서로 certificate를 검증해 양방향 identity를 확인하는 TLS 구성입니다. 유효한 chain만 보는 데서 끝내지 않고 SAN을 service account와 authorization policy에 매핑해야 합니다. Certificate 수명이 짧고 자동 회전되도록 하며 만료·폐기와 clock 문제로 통신 전체가 끊기지 않게 관찰합니다. Proxy가 TLS를 종료한다면 뒤쪽 service까지 검증된 identity를 어떻게 전달하고 신뢰할지도 명확히 하겠습니다.

## 장점

- bearer token 없이도 client identity를 확인할 수 있다.

## 단점

- 프록시 TLS 종료 지점이 많으면 identity 전달이 복잡해진다.

## 주의사항 / 실무 팁

- 허용 CA와 SAN 정책을 최소화한다.
