# API Error Code 설계 정리

## 핵심 요약

- 클라이언트가 오류별 화면과 재시도 정책을 분리할 수 있다.
- 코드를 자주 바꾸면 클라이언트 호환성이 깨진다.
- 코드 이름은 외부 계약으로 보고 변경을 통제한다.

## 개념 설명

API Error Code는 HTTP status보다 더 세밀한 실패 원인을 안정적인 문자열로 표현하는 계약이다.

코드는 도메인 의미, retry 가능 여부, 사용자 조치 가능 여부를 기준으로 정하고 내부 예외 클래스 이름을 그대로 노출하지 않는다.

## 예시

```json
{"status":409,"code":"ORDER_ALREADY_PAID","retryable":false}
```

`retryable` 같은 보조 정보가 있으면 SDK가 코드별 정책을 하드코딩하지 않아도 된다.

## 면접 답변 예시

> API error code는 HTTP status보다 세밀한 domain 실패 원인을 client가 안정적으로 분기할 수 있게 하는 외부 계약입니다. 내부 exception이나 table 이름을 그대로 노출하지 않고 사용자 조치와 retry 의미가 실제로 다른 경우에만 code를 나누겠습니다. `retryable`이 true여도 idempotency와 `Retry-After` 없이 client가 무조건 반복하지 않도록 SDK 정책을 함께 정의해야 합니다. HTTP status와 code 조합을 test하고 폐기 code는 migration 기간 동안 문서와 telemetry에 남깁니다.

## 장점

- 지원팀이 고객 문의를 코드로 검색하기 쉽다.

## 단점

- 너무 세분화하면 문서와 구현이 금방 어긋난다.

## 주의사항 / 실무 팁

- HTTP status와 code 조합을 테스트한다.
