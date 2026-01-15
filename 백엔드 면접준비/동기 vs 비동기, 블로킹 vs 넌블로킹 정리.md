# 동기 vs 비동기, 블로킹 vs 넌블로킹 정리 (Java/Spring 중심)

> **동기/비동기**는 **완료를 기다리는가**의 관점, **블로킹/넌블로킹**은 **스레드(제어권)를 멈추는가**의 관점입니다. 두 축은 독립적이며 조합될 수 있습니다.

---

## 1) 핵심 정의

### 동기(Synchronous)

- 호출자가 **작업 완료를 기다림**. 다음 단계는 **결과가 준비된 후** 진행.
- 예: 메서드 호출 → 반환값을 즉시 사용.

### 비동기(Asynchronous)

- 호출자가 **작업 완료를 기다리지 않음**. **콜백/훗날 완료 통지(Future/Promise)** 로 결과를 받음.
- 예: 작업을 큐에 등록하고, 완료되면 콜백 실행 또는 `Future` 완료.

### 블로킹(Blocking)

- I/O 등 **대기 동안 스레드가 멈춤**(제어권 반환 없음). 스레드가 **점유**되어 다른 작업 불가.

### 넌블로킹(Non‑Blocking)

- 호출 즉시 **제어권을 곧바로 반환**. 진행 가능 여부만 알려주고, 준비되지 않았으면 **다시 시도/콜백**.

> 조합 예시
>
> - **동기 + 블로킹**: 전통적 JDBC 호출(결과 올 때까지 스레드 대기)
> - **동기 + 넌블로킹**: `poll()`로 즉시 실패/재시도 루프(바쁜 대기)
> - **비동기 + 블로킹**: 비동기 제출 후 결과 `get()`으로 **대기**(결국 블로킹)
> - **비동기 + 넌블로킹**: 제출 후 **콜백/리액티브**로 완료 통지

---

## 2) 예시 코드 스케치

### 동기 + 블로킹

```java
String body = httpClient.send(request, BodyHandlers.ofString()); // 완료까지 대기
process(body);
```

### 비동기 + 넌블로킹 (Future/CompletableFuture)

```java
CompletableFuture<String> fut = httpClient.sendAsync(req, BodyHandlers.ofString());
fut.thenApply(this::process)          // 완료 후 처리
   .exceptionally(this::fallback);
```

### 비동기 제출 후 블로킹 대기(권장하지 않음)

```java
String body = fut.get(); // 여기서 블로킹
```

---

## 3) 서버 측 I/O 모델 요약

| 모델                  | 특성                          | 장점                   | 단점                          |
| --------------------- | ----------------------------- | ---------------------- | ----------------------------- |
| 스레드당 요청(블로킹) | 요청당 스레드 1개, 블로킹 I/O | 단순, 디버깅 용이      | 대량 동시접속에서 스레드 폭증 |
| 이벤트 기반(넌블로킹) | 소수 스레드 + 이벤트 루프     | 높은 동시성, 자원 효율 | 콜백/리액티브 복잡성          |
| 하이브리드            | 블로킹 API 위에 스레드풀 조합 | 단계적 도입            | 컨텍스트 전파 주의            |

---

## 4) Spring에서의 비동기 처리

### 4.1 `@Async` 기본

```java
@EnableAsync
@Configuration
class AsyncConfig {
  @Bean TaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor ex = new ThreadPoolTaskExecutor();
    ex.setCorePoolSize(10);
    ex.setMaxPoolSize(50);
    ex.setQueueCapacity(1000);
    ex.setThreadNamePrefix("async-");
    ex.initialize();
    return ex;
  }
}

@Service
class ReportService {
  @Async // 프록시가 별도 스레드에 위임
  public CompletableFuture<Report> generate(Long id) {
    // 무거운 연산/외부 I/O
    return CompletableFuture.completedFuture(new Report());
  }
}
```

- **프록시 기반**: **외부에서 호출**해야 동작(자기 자신 메서드 직접 호출은 동기 실행).

### 4.2 예외 처리

- `@Async` **void** 리턴: 예외가 **호출자에게 전파되지 않음** → `AsyncUncaughtExceptionHandler` 구성.
- `CompletableFuture` / `ListenableFuture` 리턴 시: `exceptionally`, `whenComplete` 등으로 처리.

```java
@Configuration
class AsyncExceptionConfig implements AsyncConfigurer {
  @Override
  public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return (ex, method, params) -> log.error("Async error in {}: {}", method, ex.getMessage(), ex);
  }
}
```

### 4.3 트랜잭션, 보안 컨텍스트

- 비동기 메서드의 트랜잭션은 **호출자와 분리**. 별도의 `@Transactional` 범위가 필요.
- 보안/로깅 MDC/Locale 등 컨텍스트는 **스레드 로컬**이므로 **전파 설정**(TaskDecorator, DelegatingSecurityContextRunnable 등) 필요.

### 4.4 스레드풀 설계 주의

- 코어/최대 스레드, 큐 크기, 거부 정책을 **부하 테스트로 튜닝**.
- I/O 바운드 작업: 스레드 수 ↑, CPU 바운드: 코어 수 수준이 적절.
- **블로킹 I/O**를 비동기 메서드에서 다루면 스레드가 묶일 수 있음(풀 고갈 주의).

---

## 5) 대안: 진짜 넌블로킹 스택

### 5.1 Spring WebFlux(Reactor)

- 넌블로킹 Netty + Reactive Streams(`Mono`/`Flux`) 기반, **소수 스레드로 높은 동시성**.
- JDBC 같은 **블로킹 드라이버** 접근은 별도 **전용 스케줄러로 격리** 필요.

### 5.2 Kotlin Coroutines

- 선언은 순차적이지만 런타임은 **비동기 논블로킹**. `suspend`로 표현력↑, 구조화된 동시성.
- Reactor/Coroutines 브리지 가능.

### 5.3 메시지/이벤트 기반

- 메시지 브로커(Kafka, RabbitMQ)로 **백프레셔**/비동기 처리 파이프라인 구성.

---

## 6) 선택 가이드

- **간단한 백그라운드 작업**: Spring MVC + `@Async`로 충분.
- **대량 동시 I/O(스트리밍/롱폴링)**: WebFlux/리액티브 또는 코루틴 검토.
- **CPU 바운드 작업**: 스레드풀 크기 관리 + 작업 분할(병렬 스트림/CompletableFuture).
- **데이터베이스**: 고성능이 필요하면 **커넥션 풀/쿼리 튜닝**이 우선. R2DBC 같은 넌블로킹 드라이버는 사용처에 따라 고려.

---

## 7) 체크리스트

- [ ] 비동기 메서드를 **외부에서 호출**하도록 구조화했는가(자기 호출 X)
- [ ] **예외 처리** 경로를 설계했는가(void/`Future` 각기 다름)
- [ ] **트랜잭션/보안 컨텍스트 전파**를 확인했는가
- [ ] 스레드풀/큐 크기를 **부하 기준으로 튜닝**했는가
- [ ] 블로킹 I/O를 비동기 스레드풀에 과도하게 밀어 넣지 않았는가(풀 고갈 방지)
- [ ] 진짜 넌블로킹이 필요한지(아키텍처 전환 필요성) 검토했는가

---

## 8) 요약

- **동기 vs 비동기**: 완료 기다림 여부. **블로킹 vs 넌블로킹**: 스레드를 멈추는가.
- Spring의 `@Async`는 **프록시 + 스레드풀 위임**으로 비동기를 제공하나, **예외/컨텍스트/트랜잭션**에 유의해야 합니다.
- 높은 동시성이 요구되면 **리액티브/코루틴** 같은 **논블로킹 런타임**을 고려하고, 환경에 맞춰 **정량적 튜닝**으로 마무리하십시오.
