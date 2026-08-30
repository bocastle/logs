# Log Correlation ID 정리

## 핵심 요약

- 분산 로그 검색 시간이 줄어든다.
- 외부에서 받은 id를 검증 없이 쓰면 로그 주입이나 cardinality 문제가 생긴다.
- 외부 request id는 길이와 문자 집합을 검증한다.

## 개념 설명

Log Correlation ID는 한 요청이 여러 서비스와 로그 라인을 지나도 같은 흐름으로 묶기 위한 식별자다.

gateway에서 request id를 생성하거나 검증하고, 모든 downstream 호출과 로그에 같은 id를 전달하며 trace id와 매핑한다.

## 예시

```text
X-Request-Id: req_01HY...
log request_id=req_01HY trace_id=4f5 route=/orders status=503
```

`request_id`가 응답 헤더와 로그에 같이 있어야 고객 문의에서 바로 해당 요청을 찾을 수 있다.

## 면접 답변 예시

> Correlation ID는 한 request가 여러 service를 지나며 남긴 log를 같은 흐름으로 찾기 위한 식별자입니다. Gateway에서 외부 `X-Request-Id`의 길이와 문자 집합을 검증하고, 없거나 부적절하면 새 값을 만들어 downstream과 response에 전달하겠습니다. 이 값은 인증 수단이 아니고 개인정보도 넣지 않으며, 무제한 외부 값은 log injection과 높은 cardinality를 만들 수 있습니다. Trace가 있는 구간은 trace ID도 별도 구조화 field로 남겨 고객 문의의 request ID에서 상세 span까지 연결합니다.

## 장점

- 고객이 받은 오류와 서버 로그를 연결할 수 있다.

## 단점

- 서비스 중간에서 새 id를 만들면 흐름이 끊긴다.

## 주의사항 / 실무 팁

- 없으면 gateway에서 새로 만들고 응답에 내려준다.
