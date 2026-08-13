# Liveness Probe와 Readiness Probe 정리

## 핵심 요약

- 준비 안 된 pod로 트래픽이 가는 일을 줄인다.
- liveness가 외부 DB에 의존하면 DB 장애 때 모든 pod가 재시작될 수 있다.
- liveness는 프로세스 생존 중심으로 가볍게 둔다.

## 개념 설명

Liveness Probe는 컨테이너를 재시작해야 하는지 보는 신호이고, Readiness Probe는 트래픽을 받아도 되는지 보는 신호다.

liveness는 deadlock 같은 회복 불능 상태를, readiness는 DB 연결·캐시 warm-up·migration 대기처럼 일시적으로 traffic을 빼야 하는 상태를 확인한다.

## 예시

```yaml
livenessProbe:
  httpGet: {path: /health/live, port: 8080}
readinessProbe:
  httpGet: {path: /health/ready, port: 8080}
```

readiness 실패는 pod를 재시작하지 않고 service endpoint에서 빼지만, liveness 실패는 재시작을 유도한다.

## 면접 답변 예시

> Liveness는 process를 재시작해야 하는지, readiness는 현재 traffic을 받을 수 있는지를 판단하는 별도 신호입니다. Liveness가 외부 DB 장애에 의존하면 모든 pod가 동시에 재시작될 수 있어 process 내부의 회복 불능 상태만 가볍게 확인합니다. Readiness에는 요청 처리에 꼭 필요한 준비 상태를 반영하되 shared dependency 장애에서 모든 endpoint가 빠지는 영향도 고려합니다. 부팅이 긴 application은 `startupProbe`로 liveness가 너무 일찍 작동하지 않게 합니다.

## 장점

- 멈춘 프로세스를 자동으로 재시작할 수 있다.

## 단점

- readiness가 너무 엄격하면 정상 pod도 traffic에서 빠진다.

## 주의사항 / 실무 팁

- readiness에는 traffic 처리에 필요한 dependency만 넣는다.
