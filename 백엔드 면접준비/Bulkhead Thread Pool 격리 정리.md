# Bulkhead Thread Pool 격리 정리

## 핵심 요약

- dependency별 장애 격리가 가능하다.
- pool이 너무 많으면 전체 thread 수가 과도해진다.
- pool별 active, queue, rejected count를 관측한다.

## 개념 설명

Bulkhead Thread Pool 격리는 외부 API, DB, 파일 처리 같은 작업군마다 별도 thread pool을 둬 한쪽 포화가 전체 서버를 막지 않게 하는 방식이다.

pool size, queue limit, timeout, rejection policy를 dependency별로 정하고, 공통 request thread를 오래 점유하지 않게 한다.

## 예시

```text
paymentPool: size=32 queue=64 timeout=800ms
reportPool: size=8 queue=16 timeout=5s
reject -> 503 SERVICE_BUSY
```

작업 성격이 다른 payment와 report를 같은 pool에 넣으면 느린 report가 결제 요청까지 밀어낼 수 있다.

## 면접 답변 예시

> Bulkhead thread pool은 payment와 report처럼 성격이 다른 dependency를 별도 pool과 bounded queue로 나눠 한쪽 포화가 전체 request를 막지 않게 합니다. Pool을 너무 많이 만들면 총 thread와 context switch가 늘어나므로 중요도와 blocking 특성이 실제로 다른 경계에만 적용하겠습니다. Queue가 가득 차면 timeout까지 기다리게 두지 않고 명확한 rejection과 fallback을 반환해야 합니다. Trace, deadline과 security context가 worker thread로 올바르게 전달되는지 확인하고 pool별 active, queue와 rejected count를 관찰합니다.

## 장점

- 중요 API에 우선 자원을 배정할 수 있다.

## 단점

- queue rejection 처리가 없으면 호출자는 timeout까지 기다린다.

## 주의사항 / 실무 팁

- 업무 중요도에 따라 pool과 queue를 분리한다.
