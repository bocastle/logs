# API Request Validation Layer 정리

## 핵심 요약

- 잘못된 입력이 핵심 로직에 닿기 전에 차단된다.
- 스키마만 믿고 권한이나 상태 전이를 검증하지 않으면 우회가 생긴다.
- 검증 실패 응답에 필드 경로와 code를 포함한다.

## 개념 설명

API Request Validation Layer는 controller나 handler 앞에서 입력 형식, 필수 값, 범위를 검사하는 경계다.

스키마(schema) 기반 검증으로 타입과 길이를 먼저 막고, 도메인 규칙은 서비스 계층에서 별도 오류 코드로 처리한다.

## 예시

```yaml
amount:
  type: integer
  minimum: 1
  maximum: 1000000
currency:
  enum: [KRW, USD]
```

형식 검증과 도메인 검증을 나누면 400과 409 같은 오류 의미가 선명해진다.

## 면접 답변 예시

> API request validation layer에서는 JSON 구조, 필수 값, 타입, 길이와 범위처럼 요청 자체만으로 판단할 수 있는 조건을 먼저 검사합니다. 권한, 잔액이나 상태 전이처럼 DB와 현재 사용자 문맥이 필요한 규칙은 service 계층에서 검증해 schema validation과 섞지 않겠습니다. 실패 응답에는 안정적인 error code와 field path를 주되 민감한 입력값이나 내부 예외는 그대로 노출하지 않습니다. OpenAPI와 실제 validator를 같은 contract test로 비교하고 정규식 전에는 입력 길이를 제한해 ReDoS 비용도 확인합니다.

## 장점

- OpenAPI 문서와 서버 검증을 맞추기 쉽다.

## 단점

- 검증 규칙이 문서와 따로 놀면 SDK가 틀린 요청을 만든다.

## 주의사항 / 실무 팁

- 스키마 변경은 backward compatibility 검토와 같이 진행한다.
