# API Contract Testing 정리

## 핵심 요약

- consumer 영향이 있는 변경을 조기에 발견한다.
- consumer 예제가 부족하면 중요한 경로가 검증되지 않는다.
- 필수 consumer 경로부터 계약을 작성한다.

## 개념 설명

API Contract Testing은 provider와 consumer가 합의한 요청·응답 형식을 자동 테스트로 고정하는 방식이다.

consumer pact나 OpenAPI schema를 기준으로 CI에서 breaking change를 찾고, provider 배포 전에 실제 handler 응답과 계약을 비교한다.

## 예시

```text
consumer publishes pact
provider CI verifies pact against /orders/{id}
fail on removed field or changed status
```

계약 검증이 provider CI에 있으면 배포 전에 consumer가 의존하는 필드 제거를 잡을 수 있다.

## 면접 답변 예시

> API contract testing은 consumer가 실제로 의존하는 request와 response 계약을 provider 배포 전에 검증하는 방식입니다. Consumer pact나 OpenAPI schema를 실제 handler 응답과 비교하면 field 제거, type과 status 변경 같은 breaking change를 조기에 찾을 수 있습니다. 모든 구현 세부를 고정하면 호환 가능한 field 추가까지 막을 수 있어 필요한 상호작용과 의미만 계약으로 표현하겠습니다. 인증과 시간 의존 값을 안정적으로 제어하고 핵심 consumer의 검증 결과를 배포 승인 조건에 넣습니다.

## 장점

- 문서와 실제 응답의 차이를 줄인다.

## 단점

- 계약을 너무 엄격히 만들면 호환 가능한 필드 추가도 막을 수 있다.

## 주의사항 / 실무 팁

- 추가 필드는 허용하고 제거·타입 변경을 breaking으로 본다.
