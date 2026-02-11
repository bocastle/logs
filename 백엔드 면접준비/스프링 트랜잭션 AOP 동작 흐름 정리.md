# 스프링 트랜잭션 AOP 동작 흐름 정리

## 1) 한 줄 요약

`@Transactional`이 붙은 메서드를 호출하면 **트랜잭션 AOP 프록시**가 호출을 가로채서 **트랜잭션 매니저(PlatformTransactionManager)** 로 트랜잭션을 시작/종료하고, 그 과정에서 **TransactionSynchronizationManager(ThreadLocal)** 에 커넥션(또는 EntityManager)을 바인딩하여 같은 스레드에서 동일 트랜잭션 리소스가 재사용되도록 만듭니다.

---

## 2) 핵심 구성 요소

### 2.1 트랜잭션 AOP 프록시

- `@Transactional` 대상 빈을 프록시로 감싸 **메서드 실행 전/후에 트랜잭션 경계**를 자동으로 처리합니다.
- 내부적으로는 보통 `TransactionInterceptor`(Advice)가 트랜잭션 로직을 수행합니다.

### 2.2 트랜잭션 매니저 (PlatformTransactionManager)

- 스프링이 제공하는 트랜잭션 추상화 인터페이스입니다.
- 구현체 예시
  - JDBC: `DataSourceTransactionManager`
  - JPA: `JpaTransactionManager`
- 역할
  - 시작: `getTransaction(...)`
  - 종료: `commit(...)` / `rollback(...)`
  - 리소스(커넥션/EntityManager) 확보 및 정리

### 2.3 트랜잭션 동기화 매니저 (TransactionSynchronizationManager)

- **ThreadLocal 기반 저장소**로, 트랜잭션에 묶인 리소스를 현재 스레드에 바인딩합니다.
- 효과
  - 서비스 → 리포지토리 등 호출이 여러 번 이어져도, 같은 스레드에서는 **동일 커넥션/EntityManager** 를 재사용해 트랜잭션이 유지됩니다.
  - 커넥션을 메서드 파라미터로 계속 전달할 필요가 없어집니다.

---

## 3) 전체 동작 흐름 (정상 커밋 기준)

### Step 0. 프록시 생성

- 스프링이 `@Transactional` 대상 빈을 감지해 AOP 프록시를 생성합니다.

### Step 1. 호출 진입

1. 클라이언트가 `@Transactional` 메서드를 호출합니다.
2. 실제 빈이 아니라 **프록시**가 먼저 호출을 받습니다.

### Step 2. 트랜잭션 속성 해석

3. 프록시는 트랜잭션 속성을 해석합니다.
   - 전파(Propagation), 격리(Isolation), readOnly, timeout, 롤백 규칙 등

### Step 3. 트랜잭션 시작

4. 프록시는 `txManager.getTransaction(...)`을 호출합니다.
5. 트랜잭션 매니저는 전파 규칙에 따라
   - 기존 트랜잭션이 있으면 참여(join)
   - 없으면 새 트랜잭션 생성(begin)
6. 새 트랜잭션이라면 (JDBC 기준)
   - `DataSource`에서 `Connection` 획득
   - `autoCommit=false` 등 설정
7. 트랜잭션 매니저는 해당 리소스를 `TransactionSynchronizationManager`에 **바인딩(bind)** 합니다.

### Step 4. 비즈니스 로직 수행

8. 프록시는 실제 타깃 메서드를 실행합니다.
9. 리포지토리/DB 접근 시, 프레임워크는 `TransactionSynchronizationManager`에 바인딩된 리소스를 찾아 사용합니다.

### Step 5. 커밋 및 정리

10. 타깃 메서드가 정상 종료되면 프록시는 `txManager.commit(...)`을 호출합니다.
11. 커밋 후

- `TransactionSynchronizationManager`에서 리소스 **언바인딩(unbind)**
- 커넥션 반환/종료(또는 EntityManager 정리)

---

## 4) 예외 발생 시 흐름(롤백)

- 타깃 메서드 실행 중 예외가 발생하면 프록시가 `rollback`을 결정합니다.
- 기본 롤백 규칙
  - `RuntimeException`, `Error` → 롤백
  - Checked Exception → 기본은 커밋(필요 시 `rollbackFor`로 변경)
- `txManager.rollback(...)` 수행 후 리소스 언바인딩/정리까지 진행합니다.

---

## 5) 실무에서 자주 나오는 주의사항

### 5.1 프록시 기반 AOP의 “자기 호출” 문제

- 같은 클래스 내부에서 `this.otherMethod()`로 호출하면 프록시를 거치지 않아
  `@Transactional`이 적용되지 않을 수 있습니다.

### 5.2 트랜잭션 범위 과대

- 트랜잭션을 너무 크게 잡으면 락/커넥션 점유가 길어져 성능 저하 및 데드락 확률이 증가할 수 있습니다.

### 5.3 readOnly 오해

- `readOnly=true`는 “쓰기 금지”를 강제한다기보다, 플러시 전략 등 **최적화 힌트** 성격이 큽니다.

---

## 6) 간단 도식

```
Client
  |
  v
[Tx AOP Proxy]
  |
  |  getTransaction()
  v
[PlatformTransactionManager]
  |
  |  bind(Connection/EntityManager) to ThreadLocal
  v
Target(Service/Repository)
  |
  |  정상: commit() / 예외: rollback()
  v
unbind + resource cleanup
```

### 2.2 트랜잭션 매니저 (PlatformTransactionManager)

- 스프링이 제공하는 트랜잭션 추상화.
- 대표 구현체
  - JDBC: `DataSourceTransactionManager`
  - JPA: `JpaTransactionManager`
- 역할
  - 트랜잭션 시작: 커넥션/EntityManager 확보, 트랜잭션 시작
  - 트랜잭션 종료: 커밋/롤백, 리소스 정리

### 2.3 트랜잭션 동기화 매니저 (TransactionSynchronizationManager)

- **ThreadLocal 기반 저장소**로, 현재 스레드에 트랜잭션 리소스를 바인딩합니다.
- 이를 통해 서비스/레포지토리 계층을 오가더라도 **같은 트랜잭션 커넥션을 공유**할 수 있습니다.
- 이게 없다면 메서드 호출마다 커넥션을 인자로 전달하거나, 매번 새 커넥션을 열어 트랜잭션 경계가 깨질 수 있습니다.

---

## 3) 동작 흐름(정상 커밋 기준)

### Step 0. 클라이언트가 트랜잭션 메서드 호출

- 예: `orderService.placeOrder()` (메서드에 `@Transactional` 존재)

### Step 1. AOP 프록시가 호출을 가로챔

- 프록시는 `@Transactional`의 속성(전파, 격리, readOnly, rollbackFor 등)을 해석합니다.
- 현재 스레드에 이미 트랜잭션이 있는지 확인합니다.

### Step 2. 트랜잭션 매니저가 트랜잭션을 시작

- `transactionManager.getTransaction(...)` 호출
- 내부적으로
  - 커넥션 획득 (JDBC) 또는 EntityManager 준비 (JPA)
  - `autoCommit=false` 설정 및 격리 수준 적용(필요 시)

### Step 3. 트랜잭션 리소스를 스레드에 바인딩

- 트랜잭션 매니저는 확보한 리소스를 `TransactionSynchronizationManager`에 저장합니다.
- 이후 같은 스레드에서 DB 접근이 발생하면
  - `DataSourceUtils.getConnection(...)` 같은 경로로 **바인딩된 커넥션**을 재사용합니다.

### Step 4. 실제 타깃 메서드 실행

- 서비스 → 레포지토리 호출이 이어져도 동일 커넥션(동일 트랜잭션)을 사용합니다.

### Step 5. 정상 종료 시 커밋

- 타깃 메서드가 예외 없이 종료되면 프록시가 `transactionManager.commit(...)` 호출

### Step 6. 리소스 정리

- 커밋 후
  - `TransactionSynchronizationManager`에서 리소스 언바인딩
  - 커넥션 반환/종료(커넥션 풀로 반환)

---

## 4) 예외가 발생했을 때(롤백 흐름)

1. 타깃 메서드 실행 중 예외 발생
2. 프록시가 예외 타입을 보고 롤백 여부를 판단

- 기본 규칙
  - **RuntimeException, Error** → 롤백
  - **Checked Exception** → 기본적으로 커밋 (필요하면 `rollbackFor`로 지정)

3. 롤백 결정이면 `transactionManager.rollback(...)` 수행
4. 이후 리소스 언바인딩/반환은 커밋 때와 동일하게 정리

---

## 5) 실무에서 자주 걸리는 포인트

### 5.1 프록시 기반이라 “자기 자신 호출(self-invocation)”은 트랜잭션이 안 걸릴 수 있음

- 같은 클래스 내부에서 `this.otherTransactionalMethod()`로 호출하면 프록시를 거치지 않아 AOP가 동작하지 않을 수 있습니다.

### 5.2 트랜잭션 전파(Propagation) 이해가 중요

- `REQUIRED`(기본): 있으면 참여, 없으면 새로 생성
- `REQUIRES_NEW`: 무조건 새 트랜잭션(기존 트랜잭션은 잠시 중단)

### 5.3 readOnly 트랜잭션은 “쓰기 금지”가 아니라 “최적화 힌트”에 가깝습니다

- JPA에서는 flush 동작, dirty checking 등에 영향이 있을 수 있지만,
- DB가 실제로 쓰기를 막는지는 DB/드라이버/설정에 따라 달라질 수 있습니다.

### 5.4 Graceful Shutdown과도 연결

- 종료 시점에 진행 중 트랜잭션이 남아 있으면 커밋/롤백 처리 및 커넥션 반환이 깔끔히 끝나도록 종료 시그널 처리(Graceful Shutdown)가 중요합니다.

---

## 6) 요약 그림(텍스트)

클라이언트
→ (프록시) TransactionInterceptor
→ PlatformTransactionManager.getTransaction()
→ TransactionSynchronizationManager.bindResource(connection/EntityManager)
→ 타깃 메서드 실행
→ commit 또는 rollback
→ TransactionSynchronizationManager.unbindResource(...)
→ 커넥션 반환

---

## 3) 전체 동작 흐름(순서)

### 3.1 트랜잭션 시작

1. **클라이언트가 `@Transactional` 메서드 호출**
2. **AOP 프록시**가 호출을 가로챔
3. 프록시는 `TransactionAttribute`(전파, 격리수준, readOnly, timeout, 롤백 규칙 등)를 해석
4. 프록시는 **트랜잭션 매니저**에게 `getTransaction(...)` 호출
5. 트랜잭션 매니저는
   - (트랜잭션이 없다면) `DataSource`/`EntityManagerFactory`로부터 리소스를 확보하고 트랜잭션을 시작
   - 확보된 리소스(커넥션/EntityManager)를 **TransactionSynchronizationManager에 바인딩**
6. 이후 실제 타깃 메서드(비즈니스 로직)를 실행

### 3.2 트랜잭션 참여(동일 트랜잭션 리소스 공유)

- 타깃 메서드가 레포지토리를 호출하면, 레포지토리(JdbcTemplate, JPA 등)는
  - TransactionSynchronizationManager에서 **현재 스레드에 바인딩된 리소스**를 조회
  - 동일 커넥션/EntityManager를 사용하여 쿼리를 수행

### 3.3 트랜잭션 종료

1. 타깃 메서드가 정상 종료되면 프록시는 `commit` 경로로 이동
2. 예외가 던져지면 프록시는 `rollback` 규칙에 따라 `rollback` 또는 `commit`을 결정
3. 트랜잭션 매니저는
   - 커밋 또는 롤백 수행
   - TransactionSynchronizationManager에서 리소스 바인딩을 해제(unbind)
   - 커넥션 반환/종료, EntityManager 정리
4. 프록시가 호출자에게 정상 반환 또는 예외 전파

---

## 4) 자주 헷갈리는 포인트

### 4.1 롤백 기준(기본값)

- 기본적으로 **RuntimeException / Error** 에서 롤백
- **체크 예외(checked exception)** 는 기본 설정에선 롤백하지 않음
- 필요하면 `@Transactional(rollbackFor = ...)` 로 조정

### 4.2 전파(Propagation)

- `REQUIRED`(기본): 트랜잭션이 있으면 참여, 없으면 새로 시작
- `REQUIRES_NEW`: 기존 트랜잭션을 잠시 중단하고 **항상 새 트랜잭션 시작**
- 전파 설정에 따라 위의 “트랜잭션 시작 단계”가 ‘새로 시작’이 아닐 수도 있습니다(참여만 할 수도 있음).

### 4.3 프록시 기반의 한계(셀프 인보케이션)

- 같은 클래스 내부에서 `this.someTxMethod()`처럼 **자기 자신 메서드를 호출**하면 프록시를 거치지 않아 `@Transactional`이 적용되지 않을 수 있습니다.
- 해결책: 호출 분리(다른 빈으로), AOP 설정 재검토(AspectJ weaving 등) 등

### 4.4 readOnly 트랜잭션

- `readOnly=true`는 “읽기 전용 의도”를 힌트로 주며,
  - JPA에선 flush 전략 최적화 등 효과가 있을 수 있고
  - DB/드라이버에 따라 읽기 전용 힌트를 전달하기도 합니다.
- 다만 DB 수준에서 쓰기를 100% 막아주는 보장은 환경에 따라 다릅니다.

---

## 5) 핵심만 다시 정리

- **프록시가 트랜잭션 경계를 잡고**
- **트랜잭션 매니저가 시작/종료를 수행**하며
- **TransactionSynchronizationManager가 스레드에 커넥션을 바인딩**해서
- 한 트랜잭션 동안 서비스/레포지토리 전 구간에서 **같은 커넥션(또는 EntityManager)을 공유**하도록 만든다.

### 3.2 트랜잭션 중(리소스 재사용)

- 서비스 로직이 레포지토리를 호출하면(예: `JdbcTemplate`, JPA Repository)
  - 내부에서 커넥션/EntityManager를 새로 만들지 않고
  - **TransactionSynchronizationManager에 바인딩된 리소스를 조회하여 사용**합니다.

### 3.3 정상 종료(커밋)

1. 타깃 메서드가 예외 없이 끝나면 프록시는 **트랜잭션 매니저에게 커밋**을 요청
2. 트랜잭션 매니저는 커밋 수행 후
   - 바인딩된 리소스를 해제(unbind)
   - 커넥션 반납/정리 (또는 EntityManager 종료)

### 3.4 예외 종료(롤백)

1. 타깃 메서드에서 예외가 발생하면 프록시는 롤백 대상인지 판단
2. 롤백 대상이라면 **트랜잭션 매니저에게 롤백**을 요청
3. 롤백 후에도 동일하게 리소스 해제 및 정리를 수행

---

## 4) 꼭 같이 이해하면 좋은 설정/규칙

### 4.1 롤백 규칙(기본)

- 기본적으로 스프링은 **`RuntimeException`과 `Error`에 대해 롤백**합니다.
- 체크 예외(checked exception)는 기본적으로 롤백하지 않습니다.
- 필요하면 `@Transactional(rollbackFor = Exception.class)`처럼 커스터마이징합니다.

### 4.2 전파(Propagation)

- `REQUIRED`(기본): 기존 트랜잭션이 있으면 참여, 없으면 새로 생성
- `REQUIRES_NEW`: 기존 트랜잭션을 일시 중단하고 새 트랜잭션 생성
- `SUPPORTS`, `MANDATORY`, `NESTED` 등은 상황에 따라 사용

### 4.3 readOnly

- `@Transactional(readOnly = true)`는
  - (JPA의 경우) 변경 감지 비용 감소 등의 최적화 힌트를 줄 수 있고
  - DB에 따라 read-only 트랜잭션 힌트/최적화로 이어질 수 있습니다.
  - 단, **반드시 쓰기를 막아주는 보안 장치로만 기대하면 안** 됩니다(구현/DB 설정에 따라 다름).

### 4.4 timeout

- 지정된 시간 안에 트랜잭션이 끝나지 않으면 예외를 유도해 장시간 점유를 방지합니다.

---

## 5) 자주 헷갈리는/실수 포인트

1. **Self-invocation 문제**
   - 같은 클래스 내부에서 `this.someTxMethod()`로 호출하면 프록시를 거치지 않아 `@Transactional`이 적용되지 않을 수 있습니다.

2. **프록시 적용 범위**
   - `public` 메서드에 주로 적용되며(전통적 프록시 기반 AOP), 설정/방식에 따라 private/final 등은 제약이 있을 수 있습니다.

3. **트랜잭션은 스레드에 묶인다**
   - ThreadLocal 기반이므로, 비동기 실행(다른 스레드)으로 넘어가면 같은 트랜잭션을 자동으로 공유하지 않습니다.

4. **예외를 잡아먹으면 롤백이 안 될 수 있다**
   - 예외를 캐치하고 정상 흐름으로 반환하면 프록시는 “정상 종료”로 판단해 커밋할 수 있습니다.
   - 필요하면 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 등을 고려합니다.

---

## 6) 머릿속에 그리기 쉬운 흐름

- 프록시: “트랜잭션 시작하고, 실행하고, 커밋/롤백하고, 정리”
- 매니저: “시작/종료를 구현체별(JDBC/JPA)로 수행”
- 동기화 매니저: “현재 스레드에 트랜잭션 리소스(커넥션/EM)를 보관해 전 계층에서 공유”

3. 롤백 수행 후 리소스 해제/반납

> 기본 규칙: **RuntimeException 및 Error는 롤백**, **Checked Exception은 커밋** (단, `rollbackFor`, `noRollbackFor`로 변경 가능)

---

## 4) 전파(Propagation)와 “같은 트랜잭션”의 의미

- **Propagation.REQUIRED(기본값)**
  - 이미 트랜잭션이 있으면 **참여(join)**
  - 없으면 새로 시작
- **Propagation.REQUIRES_NEW**
  - 기존 트랜잭션을 잠시 중단(suspend)하고 **새 트랜잭션을 시작**
  - 내부 작업이 독립적으로 커밋/롤백되어야 할 때 사용
- **Propagation.NESTED**(DB/드라이버 지원 필요)
  - 세이브포인트 기반의 중첩 트랜잭션

요점은, 전파 설정에 따라 TransactionSynchronizationManager에 바인딩되는 리소스의 수명/범위가 달라지고, 그에 따라 “같은 트랜잭션”이 결정됩니다.

---

## 5) 자주 헷갈리는 포인트(실무 주의사항)

1. **Self-invocation(자기 자신 호출) 문제**
   - 같은 클래스 내부에서 `this.someTxMethod()`처럼 호출하면 프록시를 거치지 않아 `@Transactional`이 적용되지 않을 수 있습니다.

2. **프록시 적용 범위**
   - 기본적으로 스프링 AOP는 프록시 기반이어서 “스프링이 관리하는 빈”에 대해서만 적용됩니다.

3. **readOnly=true의 의미**
   - “무조건 쓰기 금지”가 아니라, DB/드라이버/JPA에 힌트를 주는 성격이며, 상황에 따라 flush 최적화 등이 일어납니다.

4. **트랜잭션 경계는 짧게**
   - 외부 API 호출/대기(I/O)가 길어지면 락 점유 시간이 늘어나 병목이 될 수 있습니다.

---

## 6) 흐름을 한 눈에(간단 다이어그램)

```
Client
  |
  v
[Tx AOP Proxy]
  |  1) getTransaction()
  v
[PlatformTransactionManager]
  |  2) bind Connection/EM to ThreadLocal (TransactionSynchronizationManager)
  v
[Target Method 실행]
  |
  |  3) 정상: commit() / 예외: rollback()
  v
[PlatformTransactionManager]
  |  4) unbind + resource cleanup
  v
Client로 반환
```

---

## 5) 옵션이 흐름에 미치는 영향

### 5.1 readOnly

- 스프링이 DB를 “강제로 읽기 전용”으로 바꾸는 것만 의미하지는 않습니다.
- JPA(Hibernate)에서는 flush 동작/스냅샷 등 비용을 줄이는 최적화가 적용될 수 있습니다.
- DB 및 구현체에 따라 실제 쓰기 차단 여부는 다를 수 있으므로, **읽기 전용 조회 트랜잭션의 성격**으로 이해하는 편이 안전합니다.

### 5.2 timeout

- 트랜잭션이 일정 시간 이상 지속되면 예외로 종료시키기 위한 설정입니다.
- Graceful shutdown과 결합 시 “끝나지 않는 요청”을 어느 정도 강제로 끊는 안전장치로도 활용됩니다.

---

## 6) 자주 헷갈리는 포인트(실무 함정)

1. **Self-invocation(자기 자신 호출)**
   - 같은 클래스 내부에서 `this.someTxMethod()`처럼 호출하면 프록시를 거치지 않아 `@Transactional`이 적용되지 않습니다.

2. **프록시 생성 방식에 따른 한계(JDK 동적 프록시 vs CGLIB)**
   - 인터페이스 기반, 클래스 기반 프록시 여부에 따라 동작 제약이 달라질 수 있습니다.

3. **트랜잭션 경계가 너무 넓으면**
   - 락 점유 시간이 길어져 병목/데드락 가능성이 커집니다.

4. **예외 처리로 롤백이 안 되는 케이스**
   - 예외를 잡아서 삼켜버리면 프록시가 예외를 감지하지 못해 커밋될 수 있습니다.
   - 필요하면 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 같은 방식이 필요할 수 있습니다.

---

## 7) 흐름을 한눈에 보기(요약)

- **호출 진입**: 프록시 → 트랜잭션 매니저 `getTransaction()`
- **리소스 바인딩**: TransactionSynchronizationManager(ThreadLocal)
- **비즈니스 로직 수행**: 같은 스레드에서 리소스 재사용
- **종료**: 정상 → commit / 예외 → rollback
- **정리**: 리소스 unbind 및 반납

1. **Self-invocation(자기 자신 내부 호출)은 프록시를 거치지 않아 트랜잭션이 적용되지 않을 수 있습니다.**
   - 예: 같은 클래스 안에서 `this.someTransactionalMethod()` 호출
   - 해결: 호출 분리(빈 분리), 프록시를 통해 호출되도록 구조 변경 등

2. **프록시 기반 AOP는 기본적으로 public 메서드에 적용되는 구성이 일반적**입니다.
   - 설정/프록시 방식에 따라 다르지만, private 메서드에 기대지 않는 편이 안전합니다.

3. **트랜잭션 리소스는 “스레드에 바인딩”**됩니다.
   - `@Async`, 별도 스레드 풀, 리액티브 체인 등으로 스레드가 바뀌면 같은 트랜잭션이 자동으로 이어지지 않습니다.

4. **예외를 잡아먹으면(try/catch) 롤백이 안 될 수 있음**
   - 예외를 내부에서 처리해 밖으로 던지지 않으면 프록시가 “실패”로 인지하지 못합니다.
   - 의도적으로 롤백하려면 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 같은 전략이 필요할 때가 있습니다.

---

## 7) 흐름을 이미지로 떠올리는 방법

- 프록시가 “문지기”처럼 **들어올 때 시작**, **나갈 때 커밋/롤백**
- 트랜잭션 매니저는 실제로 DB 리소스를 만지며
- TransactionSynchronizationManager는 “현재 스레드의 트랜잭션 가방”처럼 리소스를 넣고 빼는 역할

4. **예외를 잡아먹으면(try/catch) 롤백이 안 될 수 있습니다.**
   - 프록시는 “예외가 밖으로 던져지는지”를 기준으로 롤백을 판단합니다.
   - 예외를 처리하면서도 롤백이 필요하면
     - 예외를 다시 던지거나
     - `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 같은 방식으로 롤백 마킹이 필요할 수 있습니다.

---

## 7) 정리

- **프록시가 트랜잭션 경계를 열고 닫는다.**
- **트랜잭션 매니저가 실제 커밋/롤백/리소스 관리를 담당한다.**
- **TransactionSynchronizationManager가 스레드 단위로 동일 커넥션(또는 EntityManager)을 공유하도록 만든다.**

이 3가지가 합쳐져서, 개발자는 `@Transactional`만 선언해도 서비스/레포지토리 호출 전반에 걸쳐 일관된 트랜잭션을 사용할 수 있습니다.
