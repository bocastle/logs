# gRPC deadline 전파를 설계하는 기준 정리

## 핵심 요약

- gRPC deadline은 호출이 끝나야 하는 시각이며 서버는 남은 예산 안에서 하위 작업을 완료하거나 취소해야 한다.
- 하위 RPC마다 기본 timeout을 새로 시작하지 않고 inbound deadline에서 경과 시간과 정리 여유를 뺀 값을 전달한다.
- `DEADLINE_EXCEEDED` 뒤에도 DB와 background 작업이 계속되지 않도록 취소 가능 지점을 연결한다.

## 개념 설명

gRPC deadline은 caller가 RPC 결과를 기다릴 마지막 시각을 나타낸다. Server는 context에서 남은 시간을 확인해 작업 순서와 하위 호출 예산을 정하고, 시간이 지나면 `DEADLINE_EXCEEDED`로 종료한다.

다른 서비스로 deadline을 전파할 때 구현은 네트워크에서 보낸 절대 시각을 그대로 믿기보다 남은 timeout으로 변환해 clock skew 영향을 줄일 수 있다. 하위 RPC에는 cleanup과 응답 직렬화 시간을 남긴 더 짧은 budget을 준다. 이미 시작된 외부 부작용은 단순 취소로 되돌아가지 않으므로 멱등성과 보상 경계도 필요하다.

## 예시

```text
incoming deadline: now + 800ms
db call budget: 500ms
downstream rpc budget: 650ms
return DEADLINE_EXCEEDED when budget is exhausted
```

예시의 DB와 downstream 예산은 병렬 또는 선택적 작업이라는 전제가 필요하다. 순차 호출이라면 두 값을 단순히 각각 주지 않고 실제 남은 시간을 단계마다 다시 계산해야 한다.

## 면접 답변 예시

> gRPC deadline은 caller가 응답을 기다릴 마지막 시각입니다. Server는 inbound context의 남은 시간을 확인하고 정리 여유를 뺀 더 짧은 deadline을 하위 RPC에 전달하겠습니다. 각 단계에서 기본 timeout을 새로 시작하면 caller가 포기한 뒤에도 작업이 남을 수 있습니다. Deadline 초과와 client cancel을 구분해 측정하고, DB driver와 background 작업이 실제 취소되는지와 반드시 끝내야 할 보상 작업을 따로 설계합니다.

## 장점

- 호출 체인 전체의 지연 예산을 유지하고 caller가 포기한 뒤의 낭비 작업을 줄인다.
- Deadline 초과 위치를 서비스별 지표로 비교해 병목을 찾기 쉽다.

## 단점

- Driver와 library가 취소를 무시하면 deadline 뒤에도 resource 점유가 남는다.
- 정리 작업까지 즉시 취소하면 상태 복구와 보상 처리가 빠질 수 있다.

## 주의사항 / 실무 팁

- 취소 가능한 계산과 반드시 마무리할 commit·보상 작업의 경계를 분리한다.
- `DEADLINE_EXCEEDED`, explicit client cancel과 하위 timeout을 metric에서 구분한다.
