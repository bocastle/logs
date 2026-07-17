# API 버전 deprecation 정책을 운영하는 법 정리

## 핵심 요약

- API deprecation은 기존 동작을 즉시 끊는 것이 아니라 사용 중단 권고와 전환 일정을 알리는 운영 과정이다.
- 응답의 `Deprecation`, `Sunset`과 문서 링크를 사용하되 고객 공지와 실제 client별 사용량 추적을 함께 운영한다.
- 대체 API의 기능과 계약이 준비된 뒤 충분한 migration window를 제공하고 종료 조건을 명시한다.

## 개념 설명

API 버전 deprecation 정책은 기존 계약을 당장 바꾸지 않고 더 이상 새 사용을 권장하지 않는 시점, 전환 기간, 실제 종료 시점을 관리하는 약속이다. Deprecation 중에도 해당 resource의 의미와 동작은 유지해야 한다.

RFC 9745의 `Deprecation` header는 structured date로 폐기 권고 시점을 알리고 `rel="deprecation"` 링크는 migration 문서를 가리킨다. 실제 응답 중단 시점은 `Sunset`으로 별도 표현할 수 있다. Header는 힌트이므로 개발자 포털, 담당자 공지와 client별 호출량을 함께 사용해야 한다.

## 예시

```http
Deprecation: @1782863999
Sunset: Wed, 30 Sep 2026 23:59:59 GMT
Link: </docs/deprecations/orders-v1>; rel="deprecation"; type="text/html"
Link: </api/v2/orders>; rel="successor-version"
```

Deprecation 시점은 structured field date, 실제 종료 시점은 HTTP date 형식이라는 차이에 주의한다. `successor-version` link는 대체 endpoint를 기계적으로 알려 주고, 문서에는 동작 차이, 전환 예제와 지원 연락처를 함께 제공한다.

## 면접 답변 예시

> API deprecation은 기존 동작을 바로 끊는 것이 아니라 전환을 권고하고 종료 일정을 운영하는 과정입니다. 응답에는 `Deprecation` 시점과 migration 문서 링크를 제공하고, 실제 중단 날짜가 정해졌다면 `Sunset`도 함께 사용하겠습니다. 하지만 header만으로 모든 고객에게 전달되지는 않으므로 포털 공지와 담당자 연락, client별 호출량을 같이 관리합니다. 대체 API의 계약 테스트가 끝나고 미전환 사용량이 기준 이하일 때 종료 여부를 판단합니다.

## 장점

- 미전환 client를 실제 사용량으로 찾아 우선순위에 따라 직접 지원할 수 있다.
- 지원 종료와 대체 계약을 예측 가능한 일정으로 운영할 수 있다.

## 단점

- 대체 API가 기존 기능과 성능 요구를 충족하지 못하면 migration window가 있어도 전환이 막힌다.
- 공지 채널만 믿고 실제 호출량을 보지 않으면 종료일에 중요한 client를 갑자기 중단시킬 수 있다.

## 주의사항 / 실무 팁

- 포털, 응답 header와 담당자 공지에 같은 deprecation·sunset 날짜를 사용한다.
- Migration window 동안 버전별 client 사용량과 미전환 사유를 주간 단위로 공유한다.
