# Distributed Tracing Sampling 정리

## 핵심 요약

- head sampling으로 초기 수집 비용을 제어할 수 있다.
- head sampling이 너무 낮으면 tail sampler에 도착하지 않은 드문 오류는 복구할 수 없다.
- W3C trace context와 sampled flag 전파를 통합 테스트한다.

## 개념 설명

Distributed Tracing Sampling은 분산 요청의 trace를 head sampling과 tail sampling 정책으로 선택하고 결정을 호출 체인에 전파하는 관측성 전략이다.

head sampling은 요청 시작 시 비용을 줄이고, collector의 tail sampling은 trace 완성 후 error·latency·route 규칙으로 보존을 결정한다. sampling decision은 trace context로 전파한다.

## 예시

```text
option A: SDK head sampling = parent-based 5%
option B: record candidates -> collector tail sampling
tail policy: keep error=true or latency>2s, then 1% of remaining
both: trace context and parent decision propagated end to end
```

SDK head sampling에서 버린 trace는 collector tail sampler가 복구할 수 없다. 어떤 단계가 비용을 줄이고 어떤 단계가 보존 대상을 고르는지 pipeline별로 명확히 해야 한다.

## 면접 답변 예시

> Distributed tracing sampling은 수집 비용을 제한하면서도 오류와 느린 요청처럼 중요한 trace를 남기는 전략입니다. Head sampling은 요청 시작 때 결정해 비용이 작지만 그때 버린 trace는 뒤의 tail sampler가 되살릴 수 없습니다. Tail sampling을 쓰려면 판단에 필요한 span을 collector까지 보내고 error·latency·route 정책으로 보존한 뒤 나머지만 확률적으로 줄이겠습니다. Parent-based 결정과 W3C trace context가 전 구간에 전파되는지 확인하고 drop rate, decision wait와 collector memory를 함께 관찰합니다.

## 장점

- tail sampling으로 오류와 느린 흐름을 선별 보존할 수 있다.

## 단점

- tail sampling은 완성되지 않은 trace를 보관해 collector 메모리와 지연을 증가시킨다.

## 주의사항 / 실무 팁

- tail sampling의 decision wait와 collector 메모리 상한을 같이 조정한다.
