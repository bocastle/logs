# Service Mesh 개념 정리

## 핵심 요약

- 서비스 간 mTLS와 관측성을 일관되게 적용할 수 있다.
- proxy 계층이 추가되어 지연과 운영 복잡도가 늘어난다.
- mesh 도입 전 timeout과 retry 기본값을 명시한다.

## 개념 설명

Service Mesh는 서비스 간 통신의 mTLS, retry, timeout, traffic policy, telemetry를 애플리케이션 밖 proxy 계층에서 제공하는 구조다.

sidecar나 ambient proxy가 data plane이 되고, control plane은 route, certificate, policy를 배포한다.

## 예시

```text
service A -> local proxy -> mTLS -> remote proxy -> service B
control plane pushes retry, timeout, telemetry policy
```

mesh 정책은 코드 변경 없이 통신 정책을 바꿀 수 있지만, 애플리케이션의 도메인 권한 판단을 대신하지 않는다.

## 면접 답변 예시

> Service mesh는 service 간 mTLS, traffic policy와 telemetry를 sidecar나 ambient data plane에서 공통 제공하는 구조입니다. Application code를 바꾸지 않고 timeout과 canary를 운영할 수 있지만 proxy hop의 latency와 control plane 설정 오류라는 새 장애면이 생깁니다. Mesh의 workload identity는 service 인증을 돕지만 사용자의 domain authorization까지 대신하지는 않습니다. Gateway, mesh와 application의 retry·timeout·권한 책임을 나누고 중복 retry가 폭주하지 않도록 전체 deadline과 proxy 지표를 함께 봅니다.

## 장점

- retry와 timeout 정책을 중앙에서 관리할 수 있다.

## 단점

- 잘못된 retry 정책은 장애를 증폭시킬 수 있다.

## 주의사항 / 실무 팁

- proxy 지연과 error rate를 애플리케이션 지표와 함께 본다.
