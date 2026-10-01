# API Schema Compatibility 정리

## 핵심 요약

- 버전 증가 없이 가능한 변경과 새 버전이 필요한 변경을 나눌 수 있다.
- enum 값 추가를 클라이언트가 처리하지 못하면 런타임 오류가 난다.
- schema diff를 CI에서 breaking/non-breaking으로 분류한다.

## 개념 설명

API Schema Compatibility는 요청·응답 schema 변경이 기존 클라이언트를 깨지 않는지 판단하는 기준이다.

응답 필드 추가는 대체로 안전하지만 필드 제거, 타입 변경, enum 의미 변경, 필수 요청 필드 추가는 breaking change로 본다.

## 예시

```text
compatible: add optional response field
breaking: rename id -> order_id
breaking: require new request field without default
```

필수 요청 필드 추가는 오래된 클라이언트가 값을 보낼 수 없기 때문에 가장 흔한 호환성 사고다.

## 면접 답변 예시

> API schema compatibility는 변경된 request와 response를 기존 client가 계속 처리할 수 있는지 판단하는 기준입니다. 필수 request field 추가, field 제거와 type·의미 변경은 breaking이고 response field 추가도 strict decoder를 쓰는 client에는 안전하지 않을 수 있습니다. Schema diff만 통과시키지 않고 실제 server validation과 주요 SDK의 unknown field·enum 처리 규칙을 함께 검증하겠습니다. 호환이 어려운 입력 변경은 새 version이나 단계적 feature rollout으로 진행합니다.

## 장점

- SDK 생성과 문서 검증을 자동화하기 쉽다.

## 단점

- nullable 의미 변경은 타입은 같아도 동작을 깨뜨린다.

## 주의사항 / 실무 팁

- unknown enum 값을 처리하는 클라이언트 규칙을 둔다.
