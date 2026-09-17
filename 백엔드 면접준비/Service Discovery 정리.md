# Service Discovery 정리

## 핵심 요약

- 동적으로 늘고 줄어드는 인스턴스를 자동으로 찾을 수 있다.
- DNS TTL과 connection pool이 길면 죽은 endpoint를 계속 호출할 수 있다.
- endpoint 갱신 지연과 client cache TTL을 확인한다.

## 개념 설명

Service Discovery는 클라이언트나 proxy가 호출할 서비스 인스턴스의 현재 주소와 상태를 찾는 방식이다.

DNS, registry, service mesh control plane이 healthy endpoint 목록을 제공하고, client는 TTL과 connection reuse를 고려해 갱신한다.

## 예시

```text
orders-api.default.svc -> [10.0.1.7, 10.0.2.9]
readiness false -> endpoint removed
client refreshes after DNS TTL
```

readiness와 endpoint 갱신이 연결되어야 준비되지 않은 인스턴스로 호출이 가지 않는다.

## 면접 답변 예시

> Service discovery는 배포와 scale-out으로 계속 바뀌는 service instance 주소를 client나 proxy가 찾게 하는 방식입니다. DNS, registry나 service mesh가 healthy endpoint를 제공해도 client DNS cache와 기존 connection이 오래 남으면 제거된 instance를 계속 호출할 수 있습니다. Readiness가 실제 traffic 수신 준비를 반영하게 하고 endpoint 갱신 지연, TTL과 connection drain 시간을 함께 맞추겠습니다. Discovery 장애 때 무제한 retry로 registry까지 압박하지 않고 마지막으로 알려진 endpoint 사용 범위와 신규 연결 실패 정책을 정합니다.

## 장점

- 배포와 장애 시 unhealthy endpoint를 제외할 수 있다.

## 단점

- registry 장애는 신규 연결 실패로 이어질 수 있다.

## 주의사항 / 실무 팁

- connection 재사용 정책을 배포 drain과 맞춘다.
