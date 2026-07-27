# 데이터베이스 마이그레이션 expand-contract 순서 정리

## 핵심 요약

- rolling deployment 중 schema 호환성을 유지한다.
- dual write 일부 실패는 두 컬럼을 어긋나게 한다.
- 각 단계에 row count와 불일치 검증을 둔다.

## 개념 설명

expand-contract는 구·신 코드가 함께 동작할 수 있는 확장 단계를 먼저 배포하고 데이터 전환 뒤 옛 구조를 제거하는 무중단 migration 패턴이다.

expand에서 새 컬럼을 nullable로 추가하고 dual write와 backfill을 켠 뒤 read path를 전환하며, contract에서 구 컬럼과 호환 코드를 제거한다.

## 예시

```text
expand: add new_email nullable -> deploy dual write
migrate: backfill -> compare old/new
switch: read new_email
contract: stop old write -> drop old_email
```

contract는 모든 구버전 인스턴스와 rollback 가능 기간이 끝난 뒤 실행해야 이전 코드가 새 schema에서 깨지지 않는다.

## 면접 답변 예시

> 큰 변경을 관찰 가능한 단계로 나눈다. 긴 호환 기간은 임시 코드와 저장 비용을 늘린다. contract 전에 구버전 트래픽이 0인지 확인한다.

## 장점

- 데이터 검증과 코드 전환을 독립적으로 되돌린다.

## 단점

- contract를 서두르면 구버전 코드가 실패한다.

## 주의사항 / 실무 팁

- dual write는 단일 트랜잭션 안에서 수행한다.
