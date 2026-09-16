# API Response Envelope 설계 정리

## 핵심 요약

- SDK가 응답 파싱 구조를 일관되게 만들 수 있다.
- HTTP status를 무시하고 항상 200으로 감싸면 오류 처리가 불명확해진다.
- 오류는 Problem Details와의 관계를 먼저 정한다.

## 개념 설명

API Response Envelope는 응답 본문을 `data`, `error`, `meta` 같은 공통 껍데기로 감싸는 형식 계약이다.

성공과 실패를 같은 envelope에 억지로 넣기보다 HTTP status, Problem Details, pagination metadata가 충돌하지 않게 경계를 정한다.

## 예시

```json
{"data":[{"id":"ord_1"}],"meta":{"next_cursor":"abc","request_id":"req_7"}}
```

`meta`에는 cursor와 request id처럼 본문 데이터가 아닌 처리 맥락을 담는 편이 관리하기 쉽다.

## 면접 답변 예시

> Response envelope은 성공 data와 pagination, request ID 같은 공통 metadata의 위치를 일관되게 만드는 형식입니다. 다만 모든 실패를 HTTP 200 안의 `error`로 감싸면 cache, monitoring과 client의 표준 오류 처리가 흐려집니다. 성공 envelope과 HTTP status, Problem Details 오류 형식의 관계를 먼저 정하고 `meta`에 허용할 field도 제한하겠습니다. 작은 API까지 불필요하게 장황하게 만들지 않으며 호환성 변경은 field 추가와 새 version 중 영향에 맞는 방식을 고릅니다.

## 장점

- pagination과 request id 같은 공통 정보를 담기 좋다.

## 단점

- envelope가 과하면 작은 API 응답도 장황해진다.

## 주의사항 / 실무 팁

- `data`와 `meta`에 들어갈 필드를 문서로 제한한다.
