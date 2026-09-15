# Queue Priority 처리 정리

## 핵심 요약

- 긴급 작업의 처리 지연을 줄일 수 있다.
- 낮은 우선순위 작업이 계속 굶주릴 수 있다.
- priority 부여 권한과 기준을 문서화한다.

## 개념 설명

Queue Priority 처리는 중요도가 다른 작업을 같은 큐에서 또는 별도 큐로 나눠 높은 우선순위가 먼저 처리되게 하는 설계다.

priority field, 큐 분리, weighted polling 중 하나를 선택하고 낮은 우선순위가 영원히 밀리지 않도록 aging이나 quota를 둔다.

## 예시

```text
queues: high, normal, low
poll ratio = 5:3:1
if low oldest_age > 30m -> temporary boost
```

priority는 빠른 작업부터 처리하는 기능이 아니라 업무 중요도를 반영하는 정책이다. starvation 방지가 같이 필요하다.

## 면접 답변 예시

> Queue priority는 처리 시간이 짧은 작업이 아니라 업무상 더 긴급한 작업을 먼저 처리하도록 worker 자원을 배분하는 정책입니다. High, normal, low queue를 나누고 weighted polling을 사용하면 broker의 단일 priority 구현에만 의존하지 않고 비율을 조절할 수 있습니다. 긴급 요청이 계속 들어올 때 low queue가 굶지 않도록 최소 처리 quota나 oldest age 기반 aging을 두겠습니다. 우선순위 부여 권한을 제한하고 queue별 처리량, 실패율과 oldest message age를 함께 관찰합니다.

## 장점

- 업무 중요도에 맞춰 worker 자원을 나눌 수 있다.

## 단점

- 우선순위 남용이 생기면 모든 작업이 high가 된다.

## 주의사항 / 실무 팁

- 우선순위별 oldest message age를 따로 본다.
