# RDB 페이징 쿼리 정리

## 핵심 요약

페이징 쿼리(Paging Query)는 **전체 데이터를 일정 단위로 나눠 조회/처리**해 DB와 애플리케이션의 **리소스 사용을 줄이고 처리 시간을 단축**하는 방법입니다. MySQL에서는 보통 `LIMIT`, `OFFSET`으로 구현하지만, 큰 페이지로 갈수록 성능 저하가 생길 수 있어 **No Offset(키셋/Seek) 페이징**으로 개선할 수 있습니다.

## 페이징 쿼리의 필요성

- 한 번에 전체 데이터를 가져오지 않고 **작은 단위로 나눠 조회**합니다.
- 그 결과:
  - 데이터베이스/애플리케이션의 **리소스 사용 효율 증가**
  - 로직 처리 **시간 단축**

### LIMIT, OFFSET 방식 기본 예시 (MySQL)

```sql
select *
from subscribe
limit 500
offset 0;
```

## LIMIT, OFFSET 방식의 단점

- **뒤 페이지로 갈수록 응답 시간이 길어질 수 있습니다.**
- 이유: DBMS가 지정된 `OFFSET` 만큼 **레코드를 먼저 읽고(건너뛰고)** 나서, 원하는 데이터를 가져오기 때문입니다.

## 해결 방법: OFFSET을 쓰지 않는(= No Offset) 페이징

OFFSET을 사용하지 않는 페이징은 **이전 페이지의 마지막 데이터 값**을 기준으로 다음 페이지를 조회합니다.

### 예시 테이블

```sql
create table subscribe (
   id int not null auto_increment,
   deleted_at datetime null,
   created_at datetime not null,
   primary key(id),
   key idx_deleted_at_id(deleted_at, id)
);
```

### 특정 기간 구독 해제 사용자 조회(기본 조건)

```sql
select *
from subscribe
where
    deleted_at >= ? and deleted_at < ?;
```

### 첫 페이지 조회 쿼리 예시

상황에 따라 **첫 페이지**와 **N번째 페이지**의 쿼리 모양이 달라질 수 있습니다.

```sql
select *
from subscribe
where
    deleted_at >= ? and deleted_at < ?
order by deleted_at, id
limit 10;
```

### 다음 페이지 조회 쿼리 예시 (이전 페이지 마지막 값 기반)

예를 들어, 이전 페이지의 마지막 값이 다음과 같다고 가정합니다.

- `deleted_at = '2024-01-01 00:00:00'`
- `id = 78`

그러면 다음 페이지는 아래처럼 조회할 수 있습니다.

```sql
select *
from subscribe
where
   -- deleted_at이 같은 케이스를 대응
   (deleted_at = '2024-01-01 00:00:00' and id > 78) or
   -- 마지막 데이터 이후 데이터 조회
   (deleted_at > '2024-01-01 00:00:00' and deleted_at < ?)
order by deleted_at, id
limit 10;
```

## 장점

- (No Offset 기준) **큰 페이지로 갈수록 느려지는 문제를 완화**할 수 있습니다.
- **이전 페이지 마지막 키를 기준으로 연속 조회**하므로, 불필요한 `OFFSET` 스캔을 피할 수 있습니다.

## 단점

- **정렬 기준 컬럼(예: `deleted_at`, `id`)을 명확히 설계**해야 합니다.
- 첫 페이지와 이후 페이지의 쿼리 형태가 달라질 수 있어 **구현 복잡도가 증가**할 수 있습니다.
- “이전 페이지의 마지막 값”을 전달/보관해야 하므로 **클라이언트-서버 간 인터페이스 설계**가 필요합니다.

## 주의사항 및 실무 팁

- **정렬 키는 유일성이 보장되도록 구성**하세요.
  - 예: `order by deleted_at, id`처럼 **타임스탬프 + PK** 조합으로 동점 처리.
- **인덱스 설계가 핵심**입니다.
  - 예: `(deleted_at, id)` 복합 인덱스처럼 **조회 조건 + 정렬**을 함께 고려.
- 페이지네이션 요구사항이 “무한 스크롤/더보기”에 가깝다면, **No Offset(키셋) 페이징**이 특히 잘 맞습니다.
