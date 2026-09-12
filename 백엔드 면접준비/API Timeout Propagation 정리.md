# API Timeout Propagation 정리

## 핵심 요약

- 연쇄 호출에서 tail latency를 제어하기 쉽다.
- deadline header를 신뢰 없이 외부에서 받으면 공격자가 시간을 조작할 수 있다.
- 외부 요청의 deadline은 gateway에서 검증·재작성한다.

## 개념 설명

API Timeout Propagation은 진입점에서 받은 시간 예산을 내부 서비스와 외부 호출에 계속 전달하는 설계다.

요청 context나 deadline header에 남은 시간을 담고, 하위 작업은 그보다 짧은 timeout을 사용해 상위 응답 시간 안에 정리한다.

## 예시

```text
X-Request-Deadline: 2026-07-10T10:00:02.500Z
service A remaining=700ms
service B timeout=500ms
```

절대 deadline을 전달하면 서비스마다 timeout을 새로 시작하는 일을 막을 수 있다.

## 면접 답변 예시

> Timeout propagation은 진입 request의 전체 시간 예산을 downstream 호출마다 이어서 전달하는 설계입니다. 각 service가 자기 timeout을 처음부터 다시 시작하지 않고 남은 budget보다 짧게 DB와 외부 호출 timeout을 설정해야 tail latency가 상위 deadline을 넘지 않습니다. 외부에서 받은 deadline header는 그대로 신뢰하지 않고 gateway에서 허용 범위로 검증하거나 다시 만들겠습니다. Context cancellation이 실제 query와 HTTP call까지 중단되는지 통합 테스트하고 trace에는 남은 budget과 어느 계층에서 timeout됐는지를 남깁니다.

## 장점

- 상위 요청 취소가 하위 작업까지 전달된다.

## 단점

- 하위 timeout이 너무 짧으면 정상 작업이 과하게 실패한다.

## 주의사항 / 실무 팁

- 하위 호출별 최소 budget과 fallback을 정한다.
