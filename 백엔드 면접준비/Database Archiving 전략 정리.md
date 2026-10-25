# Database Archiving 전략 정리

## 핵심 요약

- hot table과 index 크기를 줄인다.
- archive 조회 경로가 느리고 별도 도구를 요구한다.
- 복구 가능한 archive format과 schema version을 저장한다.

## 개념 설명

database archiving은 접근 빈도가 낮아진 데이터를 hot table에서 cold storage로 옮기면서 retention과 조회·삭제 가능성을 유지하는 전략이다.

시간 partition을 detach해 archive schema나 object storage로 export하고 checksum 검증 뒤 원본을 삭제하면 active index와 backup 범위를 줄인다.

## 예시

```text
cutoff=2025-01-01
detach orders_2024 -> export Parquet to cold storage
verify row count + checksum -> apply retention lock -> drop hot partition
```

archive가 법적 보존 대상이면 암호화 key, immutable retention, tenant별 삭제 예외를 storage lifecycle과 함께 설계해야 한다.

## 면접 답변 예시

> Database archiving은 오래된 data를 hot table과 index에서 cold storage로 옮기되 조회, 삭제와 복구 가능성을 유지하는 전략입니다. Time partition을 detach하고 schema version, row count와 checksum을 기록한 뒤 sample query까지 검증해야 원본을 삭제할 수 있습니다. 법적 보존과 개인정보 삭제 요구가 충돌할 수 있어 tenant·data class별 retention과 legal hold를 별도 정책으로 관리하겠습니다. Archive format과 encryption key를 장기간 읽을 수 있는지 주기적인 restore drill로 확인합니다.

## 장점

- 오래된 데이터의 storage 단가를 낮춘다.

## 단점

- 검증 전 원본 삭제는 복구 불가능한 유실을 만든다.

## 주의사항 / 실무 팁

- row count·checksum·샘플 query를 검증한다.
