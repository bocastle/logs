# DB Replication 정리 (MySQL/InnoDB 중심)

> 요약: **복제(Replication)** 는 Source(Primary)에서 발생한 변경을 **Binary Log** 기반으로 Replica(Secondary)에 전달·적용하여 **가용성, 읽기 확장성, 재해복구**를 달성하는 기술입니다. MySQL은 **Row / Statement / Mixed** 세 가지 binlog 포맷을 제공하며, 복제는 **I/O 스레드 → Relay Log → SQL 스레드** 순으로 동작합니다.

---

## 1. 복제의 목적과 효과

- **고가용성(HA)**: Primary 장애 시 Replica로 **Failover** 하여 다운타임 축소
- **읽기 확장(Read Scaling)**: 읽기 트래픽을 Replica로 분산
- **재해복구(DR)**: 원격 리전에 Replica를 두어 RPO/RTO를 개선
- **온라인 작업 완충**: 스키마 변경/백업/리포팅을 Replica에서 수행

> 주의: 복제는 기본적으로 **비동기**이며, 네트워크/부하에 따라 **지연(Lag)** 이 발생할 수 있습니다.

---

## 2. Binary Log 포맷 비교

### 2.1 Row-Based Logging (RBR)

- **원리**: 변경된 **행(row)의 실제 데이터**를 binlog에 기록
- **장점**
  - 실행 결과가 그대로 기록되어 **일관성 매우 높음**
  - 트리거/스토어드 함수/비결정적 함수 영향 최소화
- **단점**
  - 변경 행 수가 많을수록 **로그 용량 증가**
  - 대량 DML 시 I/O 부담, 네트워크 대역폭 사용량 증가
- **권장 사례**
  - **정확성 최우선** 환경, 이종(heterogeneous) 복제, 비결정적 연산이 섞이는 서비스

### 2.2 Statement-Based Logging (SBL)

- **원리**: 실행한 **SQL 문장**을 binlog에 기록
- **장점**
  - 로그 용량 **작음**, 네트워크 사용 **효율적**
- **단점**
  - `NOW()`, `UUID()`, 비결정적 함수/트리거/시스템 변수 영향으로 **재실행 결과 상이** 위험
- **권장 사례**
  - 대부분의 쿼리가 **결정적**이고 부하가 큰 **일괄 처리** 중심

### 2.3 Mixed (MBL)

- **원리**: 기본은 **Statement**, 비결정적/리스크가 있는 경우 **Row** 로 자동 전환
- **장점**: 저장공간과 일관성의 **균형**
- **단점**: 동작 판단이 내부 규칙에 의존 → **예측 난이도** 상승
- **권장 사례**: 전반적 효율을 보되 **정확성 훼손을 피하고 싶은** 일반 서비스

> 실무 기본값으로는 **Row 또는 Mixed** 가 널리 사용됩니다.

---

## 3. 복제 동작 흐름(비동기 복제)

1. **Source** 가 DML/DDL 실행 → **Binary Log** 기록
2. **Replica I/O 스레드** 가 Source 의 binlog를 읽어 **Relay Log** 로 저장
3. **Replica SQL 스레드** 가 Relay Log 를 재생하여 **데이터 적용**

- 일반적으로 네트워크/부하가 안정적이면 **수십~수백 ms** 단위로 동기화가 이뤄집니다.
- **Lag 모니터링**: `Seconds_Behind_Master`(레거시 용어), `performance_schema.replication` 뷰 등 사용

---

## 4. 복제 모드와 토폴로지

### 4.1 비동기(Async) 복제

- 기본 모드. Primary 는 커밋 시 Replica 응답을 **기다리지 않음**
- 장점: **지연 최소** / 쓰기 성능 최대화
- 단점: 장애 시 **일시적 데이터 유실(RPO>0)** 가능

### 4.2 반동기(Semi-Sync) 복제

- 커밋 시 **최소 한 개 Replica** 가 이벤트 수신(또는 Relay Log 기록)까지 **대기**
- 장점: 데이터 유실 위험 **감소**
- 단점: 네트워크/Replica 상태에 따라 **쓰기 지연** 증가

### 4.3 그룹 복제 / 멀티 프라이머리 (참고)

- **MySQL Group Replication**/**InnoDB Cluster**: 멤버십/합의로 충돌 회피, 자동 장애조치
- 장점: **내장형 고가용성**
- 단점: 구성 복잡, 워크로드 제약(충돌 가능성, 쓰기 분산 제한)

### 4.4 토폴로지

- **단일 Primary – 다중 Replica**(일반적)
- **체인(Chain) 복제**: 상위 Replica가 하위의 Source 역할(지연/복잡도 증가 주의)
- **지리적 DR**: 원거리 리전 Replica(지연·대역폭 고려)

---

## 5. 일관성과 식별: GTID vs File/Position

### GTID (Global Transaction ID)

- 트랜잭션마다 전역 유일 ID 부여 → **자동 위치 추적**, Failover/리플레이 단순화
- 운영 편의성 ↑, 복구/재구성 시 **인간 실수 감소**
- 마이그레이션/혼합 워크로드에서도 권장

### 파일/포지션 방식

- `mysql-bin.000123:4-5678` 형태로 위치 관리
- 가볍지만 **Failover 자동화**와 재동기화가 **불편/오류** 소지

---

## 6. 복제와 읽기 분산

- **읽기 전용 트래픽**을 Replica로 분산: 리포팅/검색/타임라인 등
- 주의: Replica Lag 존재 시 **강한 일관성 요구 API** 에서는 Primary 로 라우팅
  - 패턴: **Read-Your-Writes** 보장 위해 쓰기 직후 짧은 시간 Primary 고정

---

## 7. 운영 이슈와 베스트 프랙티스

### 7.1 모니터링

- **Lag**: Seconds behind, Relay/SQL thread 상태
- **스토리지/네트워크**: IOPS, fsync 지연, NIC 대역
- **binlog/relaylog** 용량, purging 정책
- 경고 임계값 설정 및 **알림** 자동화

### 7.2 구성 팁

- `binlog_format` = **ROW** 또는 **MIXED**
- `sync_binlog`, `innodb_flush_log_at_trx_commit` 로 **내구성/성능 균형**
- **GTID 모드 활성화** 권장
- Replica 에서 `read_only`, `super_read_only` 로 쓰기 보호
- 대량 DDL/DML 시 **`pt-online-schema-change`** 등 온라인 도구 고려
- **전송 압축**(`binlog_row_image`, `--compress`) 및 **로테이션** 튜닝

### 7.3 장애 대응

- Failover 자동화: Orchestrator/Proxy 기반 **프롬프팅**(가상 IP, Proxy 레벨 라우팅)
- **재동기화**: GTID 있으면 `AUTO_POSITION=1` 로 간단히 재연결
- **데이터 손상 방지**: 체크섬, 백업 검증, `read_only`/`semi-sync`로 RPO 최소화

### 7.4 보안

- 복제 전용 계정에 **최소 권한**(REPLICATION SLAVE/CLIENT)
- **TLS(SSL)** 로 채널 암호화
- 방화벽/보안 그룹으로 **소스 IP 제한**

---

## 8. Row/Statement/Mixed 선택 가이드 (요약)

| 항목      | Row               | Statement              | Mixed        |
| --------- | ----------------- | ---------------------- | ------------ |
| 일관성    | 매우 높음         | 낮을 수 있음(비결정적) | 중간~높음    |
| 로그 크기 | 큼                | 작음                   | 중간         |
| 적용 예측 | 쉬움              | 어려움(실행 맥락 영향) | 중간         |
| 권장도    | 높음(일반적 기본) | 제한적                 | 높음(균형형) |

---

## 9. 복제 지연(Lag) 완화 전략

- **Replica 스펙 상향/스토리지 튜닝**(fsync 지연/IO 큐 폭)
- **핫 파티션/대량 쓰기 분리**(샤딩, 큐잉, 배치)
- **쓰기 최적화**(작은 트랜잭션, 인덱스 정비)
- **네트워크 지연** 최소화(동일 AZ/리전, 전용 링크)

---

## 10. 마무리

- MySQL 복제는 **binlog → relaylog → 재생**이라는 단순한 모델이지만, 포맷/토폴로지/내구성 파라미터에 따라 **일관성·성능·운영 편의**가 크게 달라집니다.
- 기본값으로 **Row 또는 Mixed + GTID + 모니터링/자동화** 를 채택하고, 서비스 목적(RPO/RTO/Read-Scale)에 맞춰 **반동기/클러스터/DR** 를 단계적으로 도입하는 전략이 실무에서 가장 안정적입니다.
