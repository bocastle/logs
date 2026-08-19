# Connection Pool Saturation 정리

## 핵심 요약

- downstream 병목이 API 지연으로 나타나는 지점을 찾을 수 있다.
- pool만 키우면 downstream 포화를 더 키울 수 있다.
- active, idle, pending, timeout을 같은 대시보드에 둔다.

## 개념 설명

Connection Pool Saturation은 사용 가능한 connection이 모두 점유되어 새 요청이 대기하거나 timeout 나는 상태다.

active, idle, pending, acquisition timeout을 보고 느린 downstream, 누수, pool size 부족, 과도한 동시성을 구분한다.

## 예시

```text
active=50 idle=0 pending=120 acquire_timeout=2s
check: slow calls -> leak -> pool size -> caller concurrency
```

`pending`이 늘고 `idle`이 0이면 호출자가 connection을 기다리는 시간이 API 지연으로 번지고 있다는 뜻이다.

## 면접 답변 예시

> Connection pool saturation은 사용 가능한 connection이 모두 사용 중이라 새 요청이 대기하는 상태입니다. `active`가 상한에 붙고 `idle`은 0인데 `pending`이 늘면, pool 크기만 올리기 전에 느린 query와 connection 누수부터 확인하겠습니다. Pool을 무작정 키우면 DB 동시 부하까지 커질 수 있어서 DB가 감당할 수 있는 동시성과 호출부 backpressure를 함께 조정해야 합니다. Connection 획득 timeout은 전체 request deadline보다 짧게 두어 실패가 뒤늦게 전파되지 않게 하겠습니다.

## 장점

- pool 크기 증설과 쿼리·외부 호출 개선을 구분할 수 있다.

## 단점

- 누수를 놓치면 시간이 갈수록 회복되지 않는다.

## 주의사항 / 실무 팁

- pool size 변경 전 downstream capacity를 확인한다.
