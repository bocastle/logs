# 스레드 풀 포화 정책(Saturation Policies) 정리

## 핵심 요약

- 스레드 풀이 **포화 상태**(스레드가 `maxPoolSize`까지 증가 + 작업 대기열이 가득 참)일 때, **새 작업을 어떻게 처리할지**를 정하는 정책입니다.
- Java `ThreadPoolExecutor`에서는 포화 시 **`RejectedExecutionHandler` 구현체**가 실행됩니다.
- 기본 제공 정책은 **4가지**(AbortPolicy, CallerRunsPolicy, DiscardOldestPolicy, DiscardPolicy)이며, 필요하면 **커스텀 구현**도 가능합니다.

## 스레드 풀 포화 정책이란?

자바의 `ThreadPoolExecutor`를 기준으로, 스레드 풀 포화 정책(Saturation Policies)은 **스레드 풀이 포화 상태인 경우의 행동을 결정하는 정책**을 의미합니다.

`ThreadPoolExecutor` 설정에는 다음과 같은 요소들이 있습니다.

- `corePoolSize`: 상시 유지하는 스레드 수
- `workQueueSize`: 작업 대기열 크기
- `maxPoolSize`: 스레드를 추가할 수 있는 최대 수

스레드가 `maxPoolSize`까지 늘어나고 **대기열까지 꽉 찬 상태**를 포화 상태라고 합니다.  
이때 새로운 작업 요청이 들어오면, `RejectedExecutionHandler`의 구현체인 포화 정책이 실행됩니다.

## 포화 정책의 종류

포화 정책 종류는 `RejectedExecutionHandler`의 구현체를 기준으로 아래 4가지가 존재합니다.

### AbortPolicy

- 포화 상태일 경우, `RejectedExecutionException`을 발생시킵니다.

### DiscardPolicy

- 포화 상태일 경우, 신규 요청을 무시합니다.

### DiscardOldestPolicy

- 포화 상태일 경우, 작업 대기열에서 **가장 오래된 요청을 버리고** 신규 요청을 대기열에 추가합니다.

### CallerRunsPolicy

- 포화 상태일 경우, **요청 스레드에서 해당 작업을 실행**합니다.

## 커스텀 포화 정책

기본적으로 제공되는 포화 정책은 4가지지만, 직접 `RejectedExecutionHandler` 인터페이스를 구현하여 커스텀 포화 정책을 만들 수 있습니다.

```java
class CustomPolicy implements RejectedExecutionHandler {

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // ... 중략
    }
}
```

## 장점

- 포화 상태에서의 동작을 정책으로 분리해, **장애/부하 상황 대응 전략을 명시적으로 선택**할 수 있습니다.
- 기본 제공 정책(4가지)으로도 다양한 상황(예외 처리/요청 스레드 실행/드롭/오래된 작업 제거)을 커버할 수 있습니다.

## 단점

- 정책 선택에 따라 예외 발생, 요청 지연, 작업 유실 등이 생길 수 있어 **요구사항에 맞는 전략 설계가 필요**합니다.

## 주의사항 / 실무 팁

- “버리면 안 되는 작업”인지, “지연되어도 되는 작업”인지, “빠르게 실패시켜도 되는 작업”인지에 따라 정책을 선택하세요.
- `CallerRunsPolicy`는 요청 스레드에서 작업을 수행하므로, 트래픽이 높을 때 **요청 처리 지연**이 커질 수 있습니다.
- `DiscardPolicy`/`DiscardOldestPolicy`는 작업이 유실될 수 있으니, 유실 허용 여부와 모니터링/로깅 전략을 함께 고려하세요.
- 포화는 단순히 정책만으로 해결되지 않을 수 있으므로, `corePoolSize/maxPoolSize/workQueueSize` 조정과 함께 **병목 원인(작업 시간/외부 I/O/락 등)** 을 점검하세요.
