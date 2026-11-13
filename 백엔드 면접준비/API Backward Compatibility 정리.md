# API Backward Compatibility 정리

## 핵심 요약

- 모바일 앱처럼 업데이트가 느린 클라이언트도 안정적으로 지원한다.
- 숨은 의미 변경은 테스트로 잡기 어렵다.
- 지원 클라이언트 버전과 제거 가능 시점을 명시한다.

## 개념 설명

API Backward Compatibility는 서버가 바뀌어도 기존 클라이언트가 같은 의미로 계속 동작하게 하는 성질이다.

응답 필드 제거와 의미 변경을 피하고, 새 동작은 optional field, 새 enum 처리 규칙, version negotiation으로 점진 배포한다.

## 예시

```text
v1 response keeps total_amount
v1.1 adds amount_breakdown optional
clients ignore unknown fields
```

기존 `total_amount`를 유지한 채 선택 필드를 추가하면 오래된 클라이언트와 새 클라이언트를 동시에 지원할 수 있다.

## 면접 답변 예시

> API backward compatibility는 서버를 배포한 뒤에도 지원 대상인 기존 client가 같은 요청과 응답 의미로 계속 동작하게 하는 성질입니다. Optional field 추가도 client가 unknown field를 거부하면 깨질 수 있고 enum 값 추가는 exhaustive switch를 실패시킬 수 있어 단순히 필드 제거만 보지 않겠습니다. 지원 중인 mobile·web client fixture로 contract test를 돌리고 응답 의미와 error code 변경도 비교합니다. 새 필드는 optional로 시작해 사용률을 관측하고 제거 시점은 deprecation 공지, 호출 telemetry와 최소 지원 버전으로 결정합니다.

## 장점

- 서버 배포와 클라이언트 배포를 분리할 수 있다.

## 단점

- 호환성 유지 기간이 길면 서버 코드가 복잡해진다.

## 주의사항 / 실무 팁

- 응답 예제 기반 회귀 테스트를 둔다.
