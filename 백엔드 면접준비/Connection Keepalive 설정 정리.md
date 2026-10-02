# Connection Keepalive 설정 정리

## 핵심 요약

- 연결 생성 비용과 TLS handshake 지연을 줄인다.
- idle timeout 순서가 어긋나면 reset 오류가 생긴다.
- 네트워크 계층별 idle timeout을 표로 관리한다.

## 개념 설명

Connection Keepalive는 idle 연결을 일정 시간 유지해 매 요청마다 TCP/TLS 연결을 새로 만들지 않게 하는 설정이다.

client, load balancer, gateway, upstream의 idle timeout과 max connection age를 맞춰 중간 장비가 먼저 끊는 half-open 연결을 줄인다.

## 예시

```text
client pool max idle: 55s < load balancer idle close: 60s
load balancer upstream max idle: 55s < server keepalive close: 60s
max connection age: 30m, coordinated with deployment drain
```

각 호출자가 다음 hop이 강제로 끊기 전에 idle connection을 먼저 폐기하면 이미 닫힌 연결을 재사용하는 오류를 줄일 수 있다.

## 면접 답변 예시

> Connection keepalive는 TCP와 TLS 연결을 재사용해 handshake 비용을 줄이는 대신 여러 계층의 idle 수명을 맞춰야 하는 설정입니다. Client pool은 load balancer보다, load balancer의 upstream pool은 server보다 먼저 idle connection을 정리하게 해 이미 닫힌 socket 재사용을 줄이겠습니다. 너무 긴 연결은 특정 instance에 부하를 고정하고 rolling deploy drain을 늦출 수 있어 max connection age도 둡니다. 배포 뒤 connection reset, retry와 handshake 비율을 함께 확인합니다.

## 장점

- upstream connection 수를 예측 가능하게 관리할 수 있다.

## 단점

- keepalive가 너무 길면 오래된 connection에 부하가 쏠릴 수 있다.

## 주의사항 / 실무 팁

- connection reset과 retry 지표를 배포 후 확인한다.
