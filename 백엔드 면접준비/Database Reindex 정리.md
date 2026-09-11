# Database Reindex 정리

## 핵심 요약

- bloat page를 제거해 index 크기를 줄인다.
- 재구성 동안 기존·신규 index가 disk를 함께 쓴다.
- 작업 전 index 크기의 두 배 이상 여유를 점검한다.

## 개념 설명

REINDEX는 기존 index의 key를 table에서 다시 읽어 새 물리 구조를 만들고 bloat나 손상을 제거하는 database 유지보수 작업이다.

PostgreSQL의 REINDEX CONCURRENTLY는 임시 index를 만들고 교체해 일반 쓰기를 허용하지만 여러 transaction phase와 추가 disk를 사용한다.

## 예시

```sql
REINDEX INDEX CONCURRENTLY idx_orders_customer;
```

일반 REINDEX는 강한 lock으로 읽기·쓰기를 막을 수 있으므로 운영 table에서는 lock 수준과 concurrent 지원 여부를 확인한다.

## 면접 답변 예시

> Reindex는 table data를 다시 읽어 index의 물리 구조를 재생성해 bloat나 손상을 정리하는 작업입니다. PostgreSQL의 `REINDEX CONCURRENTLY`는 일반 write를 계속 받을 수 있지만 여러 단계와 새 index 공간이 필요하고 I/O와 replica lag를 높일 수 있습니다. 단순히 느리다는 이유만으로 실행하기보다 해당 index가 실제 사용되는지와 bloat·손상 근거를 먼저 확인하겠습니다. 작업 전 disk 여유를 확보하고 lock wait, build 진행과 replication lag에 명확한 중단 기준을 둡니다.

## 장점

- 손상된 index를 table data로 재생성한다.

## 단점

- CONCURRENTLY도 schema 변경과 충돌할 수 있다.

## 주의사항 / 실무 팁

- lock wait와 replication lag에 중단 기준을 둔다.
