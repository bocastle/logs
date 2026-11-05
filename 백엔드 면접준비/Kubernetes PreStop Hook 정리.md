# Kubernetes PreStop Hook 정리

## 핵심 요약

- rolling update 중 끊긴 요청을 줄인다.
- grace period보다 정리가 길면 강제 종료된다.
- SIGTERM 처리, readiness 실패, PreStop 순서를 통합 테스트한다.

## 개념 설명

Kubernetes PreStop Hook은 container에 TERM 신호가 전달되기 전에 실행되는 hook으로, traffic drain과 정리 작업을 시작할 시간을 준다. Pod의 termination grace period는 hook 실행 전에 이미 시작된다.

Pod가 terminating 상태가 되면 EndpointSlice 갱신과 node의 종료 절차가 진행된다. PreStop이 완료된 뒤 TERM이 전달되므로 hook 시간과 애플리케이션 종료 시간이 모두 grace period 안에 들어와야 한다.

## 예시

```yaml
lifecycle:
  preStop:
    httpGet: {path: /internal/drain, port: 8080}
terminationGracePeriodSeconds: 30
```

PreStop만 믿지 말고 readiness를 빨리 내리는 흐름이 있어야 새 요청 유입을 줄일 수 있다.

## 면접 답변 예시

> Kubernetes 종료에서는 grace period가 시작된 뒤 PreStop이 실행되고, hook이 끝난 다음 container process에 TERM이 전달됩니다. 따라서 PreStop 20초와 application shutdown 20초가 필요하면 30초 grace로는 부족합니다. EndpointSlice의 terminating 반영과 load balancer 전파에는 지연이 있을 수 있어 hook에서 drain 상태를 빠르게 켜고 새 요청을 거절하되 기존 요청은 마무리하겠습니다. PreStop은 중복 호출에도 안전하고 짧게 만들며 active request와 강제 종료를 rolling update test에서 확인합니다.

## 장점

- connection과 background worker를 정리할 시간을 준다.

## 단점

- PreStop이 실패하거나 오래 걸리면 배포가 지연된다.

## 주의사항 / 실무 팁

- terminationGracePeriodSeconds를 실제 최대 처리 시간과 맞춘다.
