# API Rate Limit Header 정리

## 핵심 요약

- 호출자가 backoff 시간을 스스로 조절할 수 있다.
- 헤더 의미가 라우트마다 다르면 클라이언트가 잘못 계산한다.
- 문서에 window 단위와 reset 기준을 명확히 적는다.

## 개념 설명

API Rate Limit Header는 호출자가 현재 한도, 남은 요청 수, reset 시점을 예측하도록 응답에 싣는 운영 계약이다.

`RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` 또는 사내 표준 헤더를 2xx와 429 응답에 일관되게 붙인다.

## 예시

```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 30
Retry-After: 30
```

429에는 `Retry-After`와 reset 정보를 함께 내려야 클라이언트가 즉시 재시도하는 일을 줄일 수 있다.

## 면접 답변 예시

> Rate-limit header는 client가 현재 limit과 remaining, reset 또는 retry 시점을 알고 요청 속도를 조절하게 하는 응답 계약입니다. Window 단위와 reset 값이 delta seconds인지 시각인지 문서에 명확히 적고 2xx와 429에서 같은 의미를 유지하겠습니다. 429에는 `Retry-After`를 함께 제공하되 client는 jitter를 더해 동시에 재시도하지 않아야 합니다. Gateway와 application 중 한 곳을 권위 있는 계산 지점으로 두고 proxy가 header를 덮어써 실제 enforcement와 어긋나지 않는지 확인합니다.

## 장점

- 지원팀이 한도 초과 원인을 응답만 보고 설명하기 쉽다.

## 단점

- 남은 quota를 민감한 내부 정책처럼 노출할 수 있다.

## 주의사항 / 실무 팁

- 429뿐 아니라 정상 응답에서도 같은 헤더를 제공한다.
