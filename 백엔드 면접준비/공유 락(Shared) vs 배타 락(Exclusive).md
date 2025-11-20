# 공유 락(Shared) vs 배타 락(Exclusive) 정리

> 동시 트랜잭션 환경에서 **정합성**을 보장하기 위한 핵심 수단이 **락(Lock)** 입니다. 본 문서는 공유/배타 락의 개념, DBMS별 구문 차이, 교착 상태(데드락)와 해결 전략, 그리고 JPA/Spring 실무 팁을 정리합니다.

---

## 1) 기본 개념

### 공유 락(Shared, S)

- **읽기 목적**의 락. 여러 트랜잭션이 **동시에 공유 락을 획득**할 수 있습니다.
- 공유 락 보유 중에는 **배타 락 획득이 거부**됩니다(쓰기 금지).
- 효과: 트랜잭션 내 **조회 일관성** 보장(동시에 누군가가 수정하지 못함).

### 배타 락(Exclusive, X)

- **쓰기 목적**의 락. **배타적**으로 자원을 점유하며, **다른 어떤 락(S/X)도 허용하지 않음**.
- 효과: 해당 레코드(또는 범위)에 대해 **수정·삭제의 원자성/일관성** 보장.

> 허용 행렬(핵심 요약)
>
> - S ↔ S: **가능** (서로 동시에 읽기)
> - S ↔ X: **불가** (쓰기 대기)
> - X ↔ X: **불가** (둘 중 하나만 보유 가능)

---

## 2) DBMS별 구문/락 동작 요약

### PostgreSQL

- `FOR SHARE` → 최신 버전에서는 `FOR KEY SHARE`/`FOR SHARE`/`FOR UPDATE`/`FOR NO KEY UPDATE` 등 **세분화된 락 모드** 제공.

```sql
-- 공유 계열
SELECT * FROM table_name WHERE id = 1 FOR SHARE;        -- 읽기 공유
SELECT * FROM table_name WHERE id = 1 FOR KEY SHARE;    -- 외래키 참조 보호
-- 배타 계열
SELECT * FROM table_name WHERE id = 1 FOR UPDATE;       -- 갱신/삭제 보호
SELECT * FROM table_name WHERE id = 1 FOR NO KEY UPDATE;-- PK/유니크 키 변경 제외 배타
```

### MySQL(InnoDB)

- MySQL 8.x: `FOR UPDATE`, `FOR SHARE` (이전 `LOCK IN SHARE MODE`는 폐지 방향).

```sql
SELECT * FROM table_name WHERE id = 1 FOR UPDATE; -- 배타
SELECT * FROM table_name WHERE id = 1 FOR SHARE;  -- 공유
```

- **넥스트키 락(Next‑Key Lock)**: 레코드 락 + **갭 락**으로 범위 보호(팬텀 억제). 인덱스 사용 여부에 따라 **잠금 범위가 크게 달라짐**.

### 공통 유의

- 단순 `SELECT`는 MVCC 스냅샷 읽기(락 X)일 수 있음. **잠금 읽기**를 하려면 `FOR UPDATE/SHARE`를 명시.
- 격리 수준/인덱스 구성/쿼리 조건에 따라 **락 범위와 대기**가 달라집니다.

---

## 3) 락의 범위와 의도 락(Intention Locks)

- **레코드(행) 락**, **페이지 락**, **테이블 락**: DB/스토리지 엔진에 따라 계층적.
- **의도 락(IS/IX)**: 하위 레벨에 S/X를 걸겠다는 **선언적 힌트**로, 상위 수준의 충돌 판정을 빠르게 합니다.
  - 예: 어떤 행에 X를 걸면 테이블에는 **IX**가 함께 표시됩니다.

---

## 4) 데드락(교착) — 언제, 왜 발생하는가

**정의**: 둘(또는 그 이상) 트랜잭션이 **서로가 보유한 락을 요구**하며 **무한 대기**하는 상태.

**전형적 시나리오**

1. A는 `id=1`에 대해 **S(또는 X)** 를 잡고, 이어서 `id=2`의 X를 기다림.
2. B는 `id=2`에 대해 **S(또는 X)** 를 잡고, 이어서 `id=1`의 X를 기다림.  
   → **순환 대기**가 성립하여 데드락.

> 락 모드가 S라 하더라도, 이후 **X로 승격**하려는 순간 상호 대기가 유발될 수 있습니다.

---

## 5) 데드락 해결/예방 전략

### 5.1 순서 일관성(Ordering)

- 모든 트랜잭션이 **항상 같은 키/리소스 순서**로 락을 획득(예: 작은 id → 큰 id).  
  → **순환 대기 고리** 차단.

### 5.2 락 범위 최소화·짧은 트랜잭션

- **수정 쿼리 인접** 위치에서만 잠금 읽기 수행, **불필요한 보유 시간 감소**.
- 사용자 입력 대기/외부 호출을 **트랜잭션 밖**으로 이동.

### 5.3 타임아웃·자동 감지

- **락 대기 타임아웃** 설정(MySQL `innodb_lock_wait_timeout`, PostgreSQL `lock_timeout`).
- DB의 **데드락 감지기**가 한 트랜잭션을 **롤백**. 애플리케이션은 **재시도 로직**을 구현.

### 5.4 인덱스와 조건 최적화

- 적절한 인덱스로 **잠금 범위 축소**(불필요한 갭 락 방지).
- 범위/카디널리티가 큰 조건의 잠금 읽기는 **경합**을 유발하기 쉬움.

### 5.5 잠금 모드 선택·일관성

- 읽기에는 `FOR SHARE`, 수정 전에는 `FOR UPDATE`를 명확히 구분.
- 불필요한 테이블 락/범위 락을 피하고, 필요한 곳에서는 **명시적으로** 사용.

### 5.6 재시도 가능 설계

- 멱등 처리, 트랜잭션 **재실행**에 안전한 서비스 계층 설계(메시지/사가 패턴도 고려).

---

## 6) JPA / Spring 실무 팁

### 6.1 비관적 락

```java
// 행 갱신/삭제 보호
entityManager
  .createQuery("select u from User u where u.id=:id", User.class)
  .setParameter("id", id)
  .setLockMode(LockModeType.PESSIMISTIC_WRITE) // FOR UPDATE
  .getSingleResult();

// 읽기 보호(공유 락)
entityManager
  .createQuery("select u from User u where u.id=:id", User.class)
  .setLockMode(LockModeType.PESSIMISTIC_READ)  // FOR SHARE (DB 매핑에 의존)
  .getSingleResult();
```

- 스프링 데이터 JPA:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select o from Order o where o.id = :id")
Optional<Order> findForUpdate(@Param("id") Long id);
```

- **주의**: 데이터베이스마다 `PESSIMISTIC_READ`의 구현이 다릅니다(실제 공유 락, 또는 스냅샷 읽기 매핑).

### 6.2 낙관적 락(대안)

- `@Version` 필드로 **동시 수정 충돌**을 감지, 충돌 시 예외 → **재시도**.
- 읽기 위주의 서비스나 경합이 낮은 시스템에 유리.

---

## 7) 예제: 무한 대기 방지 패턴(의사코드)

```sql
-- 애플리케이션 시작 시
SET lock_timeout = '5s';       -- PostgreSQL
-- 또는
SET innodb_lock_wait_timeout = 5; -- MySQL(세션 단위 가능)
```

```java
for (int attempt = 1; attempt <= MAX_RETRY; attempt++) {
  try {
    // 트랜잭션 시작
    // 1) 작은 id부터 정렬하여 잠금 읽기
    // 2) 필요한 갱신 실행
    // 커밋
    break;
  } catch (DeadlockDetected | LockTimeout ex) {
    if (attempt == MAX_RETRY) throw ex;
    backoff(attempt); // 지수 백오프 후 재시도
  }
}
```

---

## 8) 체크리스트

- [ ] **락 획득 순서**를 일관되게 강제하는가
- [ ] **인덱스**가 적절히 설계되어 **잠금 범위**가 최소화되는가
- [ ] 트랜잭션이 **짧고 응집**되어 있는가(외부 I/O/사용자 대기 제거)
- [ ] **락 타임아웃/데드락 재시도** 전략이 있는가
- [ ] 읽기/쓰기 구분에 맞는 **락 모드**를 명시적으로 사용하고 있는가
- [ ] 필요 시 낙관적 락/메시지 설계 등 **대안**을 검토했는가

---

## 9) 요약

- **공유 락(S)** 은 **동시 읽기 허용·쓰기 차단**, **배타 락(X)** 은 **모든 접근 차단**으로 독점 권한을 부여합니다.
- 데드락은 **순환 대기**로 발생하며, **자원 획득 순서 일관화**, **락 범위 최소화**, **타임아웃/재시도**로 대처합니다.
- 실제 동작은 **DBMS·격리 수준·인덱스**에 의존하므로, 목표 정합성에 맞게 **쿼리/설정/코드**를 함께 설계해야 합니다.
