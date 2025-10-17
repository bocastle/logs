# Statement vs PreparedStatement 차이점

## 문제

JDBC에서 SQL을 실행할 때 `Statement`와 `PreparedStatement` 중 어느 것을 써야 할지, 두 객체의 사용 방식·성능·보안 차이를 명확히 이해해야 합니다.  
잘못 사용하면 SQL 인젝션에 취약해지거나 성능 손해가 발생할 수 있습니다.

## 특징

### Statement

- SQL을 문자열로 만들어 실행 (`stmt.executeQuery(sql)`).
- 주로 단발성, 동적 쿼리(테이블 이름 등 SQL 구조가 자주 바뀔 때)에 사용.
- 파라미터 바인딩 기능 없음 → 직접 문자열로 값을 붙여넣음.

예시:

```java
// Statement 예시 (주의: 직접 문자열 결합)
try (Statement stmt = conn.createStatement()) {
    String sql = "SELECT * FROM users WHERE age > " + 30;
    ResultSet rs = stmt.executeQuery(sql);
    // 처리...
}
```

### PreparedStatement

- 쿼리 템플릿을 미리 준비하고 `?` 플레이스홀더에 값을 바인딩.
- JDBC 드라이버 또는 DB가 파싱/컴파일한 계획을 재사용 가능.
- 파라미터 자동 이스케이프 → SQL 인젝션 위험 대폭 감소.
- 배치 처리(batch) 및 반복 실행에 적합.

예시:

```java
// PreparedStatement 예시
String sql = "SELECT * FROM users WHERE age > ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setInt(1, 30);
    ResultSet rs = pstmt.executeQuery();
    // 처리...
}
```

## 장점

### Statement

- 단순하고 빠르게 간단한 쿼리 실행 가능(단건, SQL 구조가 동적으로 변할 때).
- DDL이나 실행 계획을 미리 재사용할 의미가 없을 때 간편.

### PreparedStatement

- **보안**: 파라미터 바인딩으로 SQL 인젝션 방지.
- **성능**: 동일 쿼리를 반복 실행하면 파싱/컴파일 비용을 줄여 성능 우수.
- **가독성/유지보수**: 파라미터 바인딩으로 코드가 깔끔해짐.
- **타입 안전성**: `setInt`, `setString` 등으로 타입 명확.
- **추가 기능**: 배치 처리, 자동 생성 키 반환.

## 단점

### Statement

- **보안 취약**: 문자열 결합 시 SQL 인젝션 위험.
- **성능 저하**: 같은 쿼리를 반복하면 매번 파싱/컴파일 비용 발생.
- 코드가 지저분해지기 쉬움 (특히 문자열 조합 시).

### PreparedStatement

- **쿼리 구조가 자주 바뀌면 비효율**: 테이블명/컬럼명이 변수라면 PreparedStatement로 파라미터 처리 불가.
- 드라이버/DB 캐시가 활성화되지 않은 경우 성능 이득이 적을 수 있음.

## 최적화 팁

- 가능한 한 동일한 SQL 템플릿 재사용(커넥션 풀과 함께 사용).
- 배치 작업 시 `addBatch()` + `executeBatch()` 사용.
- 대량 조회 시 `setFetchSize()`로 fetch 단위 조정.
- `try-with-resources`로 리소스 자동 해제.
- 드라이버 옵션(`rewriteBatchedStatements=true` 등) 활용 가능.

예시:

```java
// 배치 예시
String sql = "INSERT INTO items(name, price) VALUES(?, ?)";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    for (Item it : items) {
        pstmt.setString(1, it.getName());
        pstmt.setBigDecimal(2, it.getPrice());
        pstmt.addBatch();
    }
    int[] results = pstmt.executeBatch();
}
```

## 비교 표

```table
| 항목                | Statement                         | PreparedStatement                         |
|---------------------|------------------------------------|-------------------------------------------|
| 파라미터 바인딩     | 불가 (문자열 결합 사용)            | 가능 (`?` 플레이스홀더 + setXxx())       |
| SQL 인젝션 위험     | 높음                               | 낮음                                      |
| 반복 실행 성능      | 낮음 (매번 파싱/컴파일)            | 높음 (드라이버/DB 캐시 활용 가능)         |
| 사용 용도           | 단건/동적 SQL(구조 변경)           | 반복 실행, 배치, 안전한 파라미터 처리     |
| 타입 안전성         | 낮음 (문자열로 처리)               | 높음 (`setInt`, `setString` 등)           |
| 배치 처리 지원      | 일부 DB/드라이버에서 가능           | 표준적으로 우수                             |
| 자동 생성 키 반환   | 번거로움                           | `RETURN_GENERATED_KEYS`로 간편 지원       |
```

## 결론

일반적으로 사용자 입력을 포함하거나 반복/배치로 실행되는 쿼리는 `PreparedStatement`를 권장합니다.  
보안과 성능 모두에서 유리하며, Statement는 구조적 동적 SQL이 꼭 필요한 경우에만 제한적으로 사용하는 것이 바람직합니다.
