# NOT IN 쿼리의 문제점과 최적화 방법

백엔드 개발에서 흔히 쓰이는 `NOT IN` 쿼리는 직관적이지만, 대규모 데이터셋에서는 심각한 성능 저하를 일으킬 수 있습니다.

```sql
SELECT p
FROM Post p
WHERE p.id NOT IN :postIds
```

---

## 문제점

- **부정 조건으로 인한 풀 스캔 발생**  
  대부분의 DBMS는 `NOT IN` 조건에서 전체 테이블 스캔이나 인덱스 풀 스캔을 수행합니다.  
  → 즉, 전체 데이터를 읽고 필터링해야 하므로 효율적인 실행 계획을 세우기 어렵습니다.

- **인덱스 활용도 낮음**  
  `IN` 절은 인덱스 Range Scan을 통해 빠르게 처리할 수 있지만, `NOT IN`은 인덱스 활용이 거의 불가능합니다.

- **IN 리스트가 클 경우 오버헤드 증가**  
  대량의 값을 IN 절에 넣으면 실행 계획 생성, 파싱, 최적화 과정에서 오버헤드가 늘어납니다.

- **NULL 값 처리 문제**  
  예를 들어, 아래 쿼리는 항상 빈 결과를 반환합니다.
  ```sql
  column NOT IN (1, 2, NULL)
  ```

---

## 최적화 방안

### 1. `NOT EXISTS` 활용 (권장)

```sql
SELECT p
FROM Post p
WHERE NOT EXISTS (
    SELECT 1
    FROM Post temp
    WHERE temp.id = p.id
      AND temp.id IN :postIds
)
```

- `NOT EXISTS`는 행 단위로 평가합니다.
- 매칭되는 첫 행을 찾으면 즉시 평가를 중단하기 때문에 효율적입니다.
- DBMS가 **존재하지 않음(anti-join)** 검증에 최적화되어 있어 대규모 데이터셋에서도 안정적이고 확장성 있는 성능을 제공합니다.

---

### 2. `LEFT JOIN + IS NULL` 패턴

```sql
SELECT p
FROM Post p
LEFT JOIN (
    SELECT temp.id
    FROM Post temp
    WHERE temp.id IN :postIds
) filtered
  ON p.id = filtered.id
WHERE filtered.id IS NULL
```

- `:postIds` 집합이 작은 경우 특히 유리합니다.
- 인덱스를 효과적으로 활용할 수 있으며, PK 인덱스를 사용한 JOIN 연산이 최적화됩니다.
- 조인 후 `WHERE filtered.id IS NULL` 조건으로 제외 대상을 걸러냅니다.

---

## 추가 전략 (상황별)

### 3. 임시 테이블(또는 테이블 변수) + 인덱싱

긴 `postIds`를 임시 테이블에 적재하고 인덱스를 걸어 조인/NOT EXISTS를 수행하면 성능과 플랜 재사용성이 좋아집니다.

```sql
CREATE TEMP TABLE ExcludeIds (id BIGINT PRIMARY KEY);

-- 배치 INSERT 후
SELECT p.*
FROM Post p
LEFT JOIN ExcludeIds x ON p.id = x.id
WHERE x.id IS NULL;
```

### 4. 집합 연산자 (`EXCEPT` / `MINUS`)

DB가 지원하면 차집합 연산으로 처리하면 간단하고 효율적인 플랜이 나올 수 있습니다.

```sql
SELECT p.id FROM Post p
EXCEPT
SELECT x.id FROM ExcludeIds x;
```

### 5. 배치 분할

`IN` 리스트가 매우 크면 (수만~수십만) 적당한 크기(예: 1k~5k)로 분할하여 여러 번 처리하는 것이 더 빠를 수 있습니다.

### 6. 커버링/복합 인덱스 설계

자주 쓰이는 프로젝션 컬럼을 커버링 인덱스에 포함하면 랜덤 I/O를 줄일 수 있습니다. 조인/필터 순서에 맞춘 복합 인덱스도 고려하세요.

### 7. 빈 리스트/NULL 방지 가드

애플리케이션 레벨에서 빈 컬렉션이나 NULL 입력을 방어하세요.

```java
if (postIds.isEmpty()) {
  // 예: 제외할 게 없으면 전체 반환(혹은 쿼리 생략)
  return findAll();
}
```

---

## `NOT IN` vs `NOT EXISTS` vs `LEFT JOIN … IS NULL` 요약

| 방법                  |           성능(대규모) | 인덱스 활용 | NULL 안전성 | 권장도      |
| --------------------- | ---------------------: | ----------- | ----------- | ----------- |
| `NOT IN`              |                ❌ 불리 | 낮음        | ❌ 취약     | 지양        |
| `NOT EXISTS`          |              ✅ 안정적 | 좋음        | ✅ 안전     | 가장 권장   |
| `LEFT JOIN + IS NULL` | ✅ (제외 집합 작을 때) | 좋음        | ✅ 안전     | 상황별 사용 |

---

## 실무 체크리스트

1. `NOT IN`을 `NOT EXISTS` 또는 `LEFT JOIN … IS NULL`로 바꿔봤는가?
2. 반증/조인 키에 적절한 인덱스(PK/UK)가 있는가?
3. `:postIds`가 매우 크면 임시 테이블 + 인덱스 전략을 적용했는가?
4. NULL/빈 리스트 입력을 안전하게 처리했는가?
5. `EXPLAIN / EXPLAIN ANALYZE`로 실제 플랜을 확인했는가?
6. 필요한 열만 조회하도록 프로젝션을 최소화했는가?
7. 반복 호출 시 플랜 캐시 재사용을 고려했는가?

---

## 결론

- 정확성과 성능을 모두 잡으려면 **`NOT EXISTS`**가 1순위입니다.
- 제외 집합이 작고 키 인덱스가 튼튼하면 **`LEFT JOIN … IS NULL`**도 좋은 선택입니다.
- 긴 `IN` 리스트는 **임시 테이블 / 배치 분할**로 DB가 최적화하기 쉬운 형태로 바꾸세요.
- 마지막으로 항상 **`EXPLAIN(ANALYZE)`**로 실행 계획을 확인하고 튜닝하세요.
