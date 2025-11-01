# JPA 엔티티 매니저(EntityManager) 정리

> 본 문서는 JPA/Hibernate와 Spring 환경을 전제로, **영속성 컨텍스트**와 **EntityManager**의 역할을 체계적으로 설명합니다.

---

## 1) 영속성 컨텍스트란

- **정의**: 엔티티를 **영구 저장**하고 관리하는 1차 캐시 영역입니다.
- **핵심 기능**
  - **1차 캐시**: 동일 트랜잭션 내에서 같은 엔티티를 조회할 때 DB 재조회 없이 캐시를 재사용합니다.
  - **동일성 보장(Identity)**: 같은 PK는 항상 **동일 객체**를 반환합니다.
  - **쓰기 지연(Write-Behind)**: `persist/remove` 시점에 즉시 DB 반영 대신 **SQL을 버퍼링**했다가 `flush` 타이밍에 일괄 전송합니다.
  - **변경 감지(Dirty Checking)**: 트랜잭션 커밋/flush 시점에 **스냅샷과 현재값을 비교**하여 변경된 필드에 대해 DML을 생성합니다.

> 영속성 컨텍스트에 **엔티티를 넣고 빼는** 모든 작업을 수행하는 조정자가 **EntityManager**입니다.

---

## 2) EntityManager의 역할

- **생애주기/상태 전환**: `persist`, `merge`, `remove`, `detach`, `clear`, `close`
- **조회**: `find`, `getReference`(프록시), 1차 캐시 우선 → DB
- **쓰기 지연/동기화**: `flush`로 SQL 버퍼를 DB에 전송, `FlushMode`로 타이밍 제어
- **갱신 무효화**: `refresh`로 DB 상태를 재적재(캐시 무시)
- **쿼리 실행**: `createQuery`(JPQL/Criteria), `createNativeQuery`(SQL), `setParameter`, `getResultList`
- **락/동시성**: `lock`, `LockModeType`(OPTIMISTIC/OPTIMISTIC*FORCE_INCREMENT/PESSIMISTIC*\*)
- **캐스케이드/고아객체**: 연관 필드의 `cascade`, `orphanRemoval` 해석 및 전파

> **스레드-세이프하지 않습니다.** 스프링에서는 보통 트랜잭션 범위에 묶인 **프록시 EntityManager**를 주입받아 요청/트랜잭션 단위로 안전하게 사용합니다(`@PersistenceContext` 또는 스프링 부트 자동설정).

---

## 3) 엔티티의 4가지 상태와 전환

### ① 비영속(Transient)

- **정의**: 새로 `new`로 생성했으나 **아직 영속성 컨텍스트에 관리되지 않는** 상태.
- **특징**: DB와 무관, 1차 캐시에도 없음.

```java
Member m = new Member("산초"); // 비영속
```

### ② 영속(Managed)

- **정의**: 영속성 컨텍스트가 **관리 중**인 상태.
- **특징**: 변경 감지 대상, 동일성 보장, 쓰기 지연 적용.

```java
em.persist(m);                // 영속 전환 (INSERT SQL은 보통 flush/commit 시 전송)
Member found = em.find(Member.class, 1L); // 캐시 → DB 조회
```

### ③ 준영속(Detached)

- **정의**: 한때 영속이었으나 **컨텍스트에서 분리**된 상태.
- **특징**: 변경 감지 비대상, 지연 로딩 불가(프록시 초기화 시 예외 가능).

```java
em.detach(m); // 특정 엔티티만 분리
em.clear();   // 전체 영속성 컨텍스트 비우기
em.close();   // 컨텍스트 종료(모든 엔티티 준영속)
```

### ④ 삭제(Removed)

- **정의**: 삭제 예약 상태. flush/commit 시 **DELETE** 수행.

```java
em.remove(m); // 삭제 예약
```

#### 상태 복귀/합치기: `merge`

- 준영속/비영속 객체의 상태를 **새 영속 인스턴스**에 복사하여 **영속 상태**로 돌려줍니다.
- 주의: `merge`는 **새 인스턴스(영속)** 를 반환하며, **원본**은 여전히 준영속입니다.

```java
Member detached = ...;           // 준영속
Member managed = em.merge(detached); // managed는 영속, detached는 그대로
```

---

## 4) 트랜잭션과 flush

- **트랜잭션 커밋** 시 JPA는 **자동으로 flush**를 호출하여 SQL 버퍼를 DB에 반영하고 커밋합니다.
- `flush()`는 커밋이 아니며, **동기화만 수행**합니다.
- 기본 `FlushModeType.AUTO`: 쿼리 실행 전/커밋 시 필요에 따라 자동 flush.  
  `FlushModeType.COMMIT`: 커밋 시에만 flush(특정 케이스 성능 최적화에 활용).

---

## 5) Spring에서의 사용 패턴

- **주입**: `@PersistenceContext` 또는 `@Autowired EntityManager`(스프링 부트)  
  스프링은 트랜잭션 범위로 **프록시**를 주입하여, 각 트랜잭션마다 실제 EM을 할당합니다.
- **트랜잭션 경계**: `@Transactional` 메서드 안에서 EM 사용. 읽기 전용이면 `@Transactional(readOnly = true)`.
- **EntityManagerFactory**: EM을 생성하는 팩토리. **애플리케이션 단일/소수** 빈으로 관리.

---

## 6) 자주 쓰는 메서드 요약표

| 메서드                   | 요약                                               | 비고                       |
| ------------------------ | -------------------------------------------------- | -------------------------- |
| `persist(entity)`        | 엔티티를 **영속**으로 전환(INSERT 예약)            | 식별자 전략/생성 시점 주의 |
| `find(type, id)`         | PK로 조회(1차 캐시 우선)                           | 없으면 `null`              |
| `getReference(type, id)` | **프록시** 반환, 지연로딩                          | 접근 시 초기화             |
| `remove(entity)`         | **삭제 예약**                                      | flush/commit 시 DELETE     |
| `merge(entity)`          | 준영속/비영속의 상태를 **새 영속 인스턴스**로 병합 | 반환값 사용 필수           |
| `detach(entity)`         | 해당 엔티티만 **준영속** 전환                      | 변경 감지 제거             |
| `clear()`                | 컨텍스트 **전체 비우기**                           | 신중히 사용                |
| `flush()`                | 쓰기 지연 SQL **DB 반영**                          | 커밋 아님                  |
| `refresh(entity)`        | DB 기준으로 **재로딩**                             | 캐시/변경 내용 무시        |
| `createQuery(...)`       | JPQL/Criteria 실행                                 | 타입세이프는 `TypedQuery`  |

---

## 7) 모범 사례와 주의점

- **엔티티 변경은 영속 상태에서만**: 준영속 수정 후 반영하려면 `merge` 결과를 사용하십시오.
- **Equals/HashCode**: 식별자 기반 구현 권장(생성 전/영속 전 상태 고려). 컬렉션 키로 영속 엔티티 사용 시 주의.
- **지연 로딩 N+1**: 서비스 계층에서 필요한 데이터는 **명시적 fetch**(fetch join/EntityGraph/배치 페치)로 로딩을 완료하십시오.
- **OSIV 설정**: 뷰에서 지연 로딩 접근은 예측하기 어려운 쿼리를 유발할 수 있습니다. 서비스에서 데이터 완결.
- **트랜잭션 경계 외 사용 금지**: 읽기에도 트랜잭션 경계가 필요(일관성/플러시/락).
- **스레드-세이프 아님**: 멀티스레드 공유 금지. 요청/트랜잭션 단위로 할당.

---

## 8) 요약

- **EntityManager는 영속성 컨텍스트의 조정자**로, 엔티티의 상태 전환/조회/쓰기 지연/변경 감지/쿼리 실행을 담당합니다.
- 엔티티는 **비영속 → 영속 → 준영속/삭제**로 상태가 변하며, 올바른 시점에 `persist/merge/remove/detach/flush`를 사용해야 합니다.
- 스프링에서는 트랜잭션 범위의 EM 프록시를 주입받아 안전하게 사용하며, 성능/일관성을 위해 **fetch 전략과 flush 타이밍**을 설계해야 합니다.
