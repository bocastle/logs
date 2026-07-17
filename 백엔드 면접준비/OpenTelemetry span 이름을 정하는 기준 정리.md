# OpenTelemetry span 이름을 정하는 기준 정리

## 핵심 요약

- Span 이름은 작업을 알아볼 수 있으면서도 집계 가능한 낮은 cardinality를 유지해야 한다.
- HTTP server span은 method와 `http.route` template을 사용하고 실제 ID나 전체 URL을 이름에 넣지 않는다.
- 세부 값은 필요성과 민감도를 검토해 attribute로 기록하고 수집 backend의 cardinality 비용도 확인한다.

## 개념 설명

OpenTelemetry span 이름은 trace 안에서 수행한 작업을 짧게 나타낸다. Trace 화면에서 사람이 읽는 용도뿐 아니라 이름별 지연과 오류를 집계하는 데 사용되므로 요청마다 달라지지 않는 안정적인 규칙이 필요하다.

HTTP server span은 `GET /orders/{orderId}`처럼 method와 route template을 사용한다. 계측 시점에 route를 아직 알 수 없다면 나중에 이름을 갱신하거나 method만 사용하고 실제 path를 기본 이름으로 삼지 않는다. DB와 messaging도 semantic convention의 낮은 cardinality operation과 destination을 우선한다.

## 예시

```text
good: GET /orders/{orderId}
bad:  GET /orders/123456
attribute: order.id=123456 only when safe
```

실제 주문 ID를 이름에서 제외하면 주문마다 새로운 시계열이 생기지 않는다. `order.id`도 무조건 수집하지 않고 장애 분석 필요성, 개인정보와 backend indexing 비용을 검토해 제한한다.

## 면접 답변 예시

> Span 이름은 작업을 알아볼 수 있으면서 집계 가능한 낮은 cardinality를 유지해야 합니다. HTTP server span은 method와 실제 path가 아닌 `http.route` template을 사용하겠습니다. 주문 ID 같은 값은 이름에 넣지 않고, attribute로도 꼭 필요하고 안전한 경우에만 제한적으로 남깁니다. 자동 계측과 수동 span이 서로 다른 규칙을 만들지 않도록 semantic convention과 실제 backend의 cardinality 지표를 함께 확인합니다.

## 장점

- Route와 operation별 latency·오류 집계를 안정적으로 만들 수 있다.
- 여러 서비스와 계측 라이브러리에서 trace를 비슷한 이름으로 읽을 수 있다.

## 단점

- 이름이 너무 추상적이면 trace에서 실제 작업을 구분하기 어렵다.
- 반대로 ID와 URL을 넣으면 cardinality와 저장·조회 비용이 빠르게 늘어난다.

## 주의사항 / 실무 팁

- 세부 값은 장애 분석에 필요한지와 민감정보 여부를 검토한 뒤 attribute로 제한해 넣는다.
- 자동 계측 span 이름, `http.route` 설정 시점과 backend의 고 cardinality 경고를 함께 점검한다.
