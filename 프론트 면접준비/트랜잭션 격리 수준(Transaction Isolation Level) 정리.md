# 트랜잭션 격리 수준(Transaction Isolation Level) 정리

> 동시 트랜잭션 환경에서 **정합성(Consistency)** 과 **성능(Concurrency)** 의 균형을 결정하는 핵심 설정입니다. 격리가 높을수록 정합성은 강하지만, 동시성이 낮아질 수 있습니다.

---

## 1) 격리 수준 개요

SQL 표준 기준 네 가지 격리 수준이 존재합니다.

1. **READ UNCOMMITTED**
2. **READ COMMITTED**
3. **REPEATABLE READ**
4. **SERIALIZABLE**

일반적으로 **낮은 수준 → 높은 수준** 순서로 정합성이 강해지고 동시성은 낮아집니다.

---

## 2) 각 수준별 정의와 특징

### 2.1 READ UNCOMMITTED

- **정의**: 커밋되지 않은 다른 트랜잭션의 변경도 읽을 수 있습니다.
- **장점**: 동시성 최상.
- **단점/위험**: **Dirty Read / Non-Repeatable Read / Phantom Read** 모두 발생 가능.
- **권장도**: 거의 사용하지 않습니다(비즈니스 정합성에 매우 취약).

### 2.2 READ COMMITTED

- **정의**: **커밋된 데이터만** 읽습니다. 읽는 순간의 가장 최근 커밋 데이터를 반환.
- **보장**: **Dirty Read 방지**.
- **발생 가능 문제**: **Non-Repeatable Read, Phantom Read**.
- **비고**: 많은 DB의 **기본 격리 수준**(예: PostgreSQL). **읽기-잠금 경쟁이 적당**해 운영에 널리 사용.

### 2.3 REPEATABLE READ

- **정의**: 한 트랜잭션 내에서 **같은 행(row)을 여러 번 읽어도** 항상 **동일한 스냅샷**을 봅니다.
- **보장**: **Dirty Read / Non-Repeatable Read 방지**.
- **발생 가능 문제**: **Phantom Read**(새 행의 삽입/삭제로 결과 집합이 달라짐).
- **비고**: MySQL(InnoDB) 기본값. DB 구현(MVCC+갭 락)에 따라 **팬텀 억제** 여부가 달라질 수 있습니다(아래 참고).

### 2.4 SERIALIZABLE

- **정의**: 모든 트랜잭션이 **직렬(serial)로 실행된 것과 동일한 결과**를 보장하는 **가장 강력한 격리**.
- **보장**: **Dirty / Non-Repeatable / Phantom Read 모두 방지**.
- **단점**: **동시성 저하**(락 경합 증가 또는 충돌 시 재시도 필요).
- **비고**: PostgreSQL은 스냅샷 격리+검증으로 **직렬화 오류를 발생**시켜 **트랜잭션 재시도**를 요구합니다. MySQL은 범위 락(Next-Key Lock 등)으로 동작.

---

## 3) 이상 현상(Anomalies) 정의

- **Dirty Read**: **미커밋 변경**을 읽음. 이후 롤백 시 **읽은 값이 사라지면서** 결과 왜곡.
- **Non-Repeatable Read**: 한 트랜잭션이 **같은 행을 두 번 읽을 때 값이 달라지는** 현상(다른 트랜잭션이 사이에 **갱신/삭제**).
- **Phantom Read**: **같은 조건의 쿼리를 반복**했는데 **행의 수(결과 집합)가 달라지는** 현상(다른 트랜잭션의 **삽입/삭제**).

> 요약: Dirty=미커밋 읽기, Non-Repeatable=**같은 행**의 값이 바뀜, Phantom=**결과 집합**이 바뀜.

---

## 4) 격리 수준 vs 이상 현상 표

| 격리 수준        | Dirty Read | Non-Repeatable Read | Phantom Read |
| ---------------- | :--------: | :-----------------: | :----------: |
| READ UNCOMMITTED |    발생    |        발생         |     발생     |
| READ COMMITTED   |    방지    |        발생         |     발생     |
| REPEATABLE READ  |    방지    |        방지         |  **발생\***  |
| SERIALIZABLE     |    방지    |        방지         |     방지     |

\* **DB 구현 차이**: InnoDB는 REPEATABLE READ에서 **Next-Key Lock(갭 락+레코드 락)** 으로 **팬텀을 억제**할 수 있습니다. 하지만 **읽기 방식**(순수 MVCC 스냅샷 읽기 vs 잠금 읽기), **인덱스 여부**, **쿼리 형태**에 따라 동작이 달라질 수 있습니다.

---

## 5) DBMS별 구현 노트

### 5.1 MySQL(InnoDB)

- **기본**: REPEATABLE READ.
- **MVCC** + **언두 로그**로 스냅샷 읽기 구현.
- **잠금 읽기**(`SELECT ... FOR UPDATE / LOCK IN SHARE MODE`) 시 **레코드 락/갭 락/넥스트키 락**이 동원되어 **팬텀 방지**.
- 단순 `SELECT`는 스냅샷 읽기(락 없음)로 동작하므로 **동시성 높음**.

### 5.2 PostgreSQL

- **기본**: READ COMMITTED.
- **REPEATABLE READ** = **스냅샷 격리**(한 트랜잭션 동안 일관 스냅샷). 팬텀은 일부 케이스에서 발생 가능.
- **SERIALIZABLE**: 직렬성 검사로 **Serialization Failure**(SQLSTATE 40001)를 발생시켜 **재시도 요구**.

### 5.3 Oracle

- 전통적으로 **READ COMMITTED** 기본. **Serializable** 모드 지원. 강력한 일관성 모델과 **락 전략**이 상이.

> 결론: **이론적 표준**과 **제품별 실제 동작**에는 차이가 있으니, **대상 DB의 문서/락 동작**을 반드시 확인해야 합니다.

---

## 6) 선택 가이드(실무 관점)

- **기본 선택**: 읽기가 많고 일관성이 과도하게 엄격할 필요가 없다면 **READ COMMITTED**(또는 MySQL 기본값인 **REPEATABLE READ**)가 일반적입니다.
- **집계/정산 등 무결성 중요 구간**: **SERIALIZABLE** 또는 **잠금 읽기**(범위 잠금, `SELECT ... FOR UPDATE`)를 부분적으로 사용.
- **비즈니스 불변식**(유니크 제약/계좌 잔액 등)은 **DB 제약 조건/유니크 인덱스/외래키**로 보강하고, 필요 시 **낙관적 락(버전 필드)** 를 병행.
- **재시도 전략**: SERIALIZABLE/낙관적 락 충돌은 **재시도 로직**을 준비(지수 백오프 등).

---

## 7) JPA/Spring 설정 팁

- 메서드별 격리 지정:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void serviceMethod() { ... }
```

- 전역 기본값은 **DB 드라이버/데이터소스 설정**을 따르는 경우가 많습니다.
- **락 옵션**:

```java
em.createQuery("select a from Account a where a.id=:id", Account.class)
  .setLockMode(LockModeType.PESSIMISTIC_WRITE) // SELECT FOR UPDATE
  .getSingleResult();
```

- **낙관적 락**: `@Version` 필드로 병행 수정 감지(충돌 시 예외 → 재시도).

---

## 8) 안티패턴과 주의

- 전역을 **SERIALIZABLE** 로 고정 → **과도한 경합/타임아웃** 유발.
- **인덱스 없는 범위 쿼리**에서 잠금 읽기 사용 → **광범위 갭 락**으로 경합 폭증.
- **긴 트랜잭션**(사용자 입력 대기 포함) → 락 보유 시간 증가, 교착/타임아웃 위험.
- 팬텀을 막기 위해서는 **잠금 읽기(범위 잠금)** 또는 **직렬화 수준**을 **부분 적용**하는 것이 효율적.

---

## 9) 요약

- 격리 수준은 **Dirty / Non-Repeatable / Phantom** 같은 이상 현상을 **어느 수준까지 허용/방지**할지 결정합니다.
- 표준 표는 방향성을 제시하지만, **DBMS별 구현(락/MVCC/검증 방식)** 에 따라 **세부 동작**이 다릅니다.
- 운영에서는 **READ COMMITTED/REPEATABLE READ**를 기본으로, **핵심 구간에만 잠금 읽기/직렬화**를 선택적으로 적용하는 전략이 일반적입니다.
