# Database Migration Rollback 정리

## 핵심 요약

- code rollback이 가능한 schema window를 만든다.
- 파괴적 migration은 되돌려도 데이터를 복구하지 못한다.
- 모든 migration의 backward-compatible 기간을 정한다.

## 개념 설명

database migration rollback은 실패한 schema 변경 뒤 이전 application이 계속 동작하도록 호환 상태를 복원하는 절차다.

파괴적 DDL을 즉시 되돌리기보다 backward-compatible expand 상태를 유지하고 code rollback 또는 forward fix를 선택하는 roll-forward 중심 전략이 안전하다.

## 예시

```text
add column -> code deploy 실패
action: code rollback, 새 nullable column 유지
fix forward 검증 뒤 다음 release에서 contract
```

DROP COLUMN이나 lossy type 변환은 rollback SQL로 데이터를 복원할 수 없으므로 사전 backup·shadow column·호환 기간이 필요하다.

## 면접 답변 예시

> Database migration rollback은 단순히 down SQL을 실행하는 것이 아니라 이전 application이 새 schema에서도 동작할 호환 window를 만드는 일입니다. Nullable column 추가 같은 expand 단계에서는 code를 되돌리고 schema는 남겨 두는 편이 급한 역 DDL과 추가 lock을 피할 수 있습니다. Column drop이나 lossy type 변환은 rollback SQL만으로 data를 복원할 수 없어 shadow copy, backup과 검증된 restore 절차가 필요합니다. Migration마다 code rollback과 roll-forward 판단 기준을 정하고 destructive contract는 호환 기간이 끝난 뒤 실행합니다.

## 장점

- 실패 시 추가 DDL lock을 피한다.

## 단점

- down migration이 새 코드가 쓴 값을 이해하지 못할 수 있다.

## 주의사항 / 실무 팁

- 파괴 작업 전 복구 가능한 copy를 만든다.
