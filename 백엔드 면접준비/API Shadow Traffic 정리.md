# API Shadow Traffic 정리

## 핵심 요약

- 실제 트래픽 분포로 새 API 동작을 검증할 수 있다.
- 쓰기 요청을 그대로 shadow하면 중복 부작용이 생긴다.
- shadow 대상은 읽기 전용 또는 격리된 저장소를 사용한다.

## 개념 설명

API Shadow Traffic은 실제 요청을 사용자 응답에는 영향을 주지 않는 새 버전이나 후보 서비스로 복제해 검증하는 방식이다.

gateway나 proxy가 요청을 원본으로 보내면서 비동기로 shadow target에도 보내고, 응답 차이와 latency만 관측한다.

## 예시

```text
client -> gateway -> stable service -> response
                 \-> shadow service -> diff metric only
```

shadow 응답은 사용자에게 반환하지 않으므로 후보 서비스의 부작용 쓰기를 막아야 한다.

## 면접 답변 예시

> Shadow traffic은 실제 request를 stable service와 후보 service에 함께 보내되 후보 응답은 사용자에게 반환하지 않는 검증 방식입니다. Candidate 호출은 원본 response latency와 backpressure에 영향을 주지 않도록 비동기·제한된 비율로 복제하겠습니다. Write request는 중복 부작용을 막기 위해 읽기 전용 mode나 격리된 data store에서 처리하고 auth token과 개인정보 복제 범위도 최소화해야 합니다. Response diff의 정규화 규칙과 허용 오차를 먼저 정하고 shadow saturation과 비용을 별도로 관찰합니다.

## 장점

- 사용자 영향 없이 latency와 오류율을 비교할 수 있다.

## 단점

- 개인정보를 추가 환경으로 복제하면 규정 위험이 커진다.

## 주의사항 / 실무 팁

- 응답 diff 기준과 허용 오차를 미리 정한다.
