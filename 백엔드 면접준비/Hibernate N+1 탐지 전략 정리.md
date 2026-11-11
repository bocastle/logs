# Hibernate N+1 탐지 전략 정리

## 핵심 요약

- 반복 SQL을 request 단위로 빠르게 찾는다.
- 무조건 fetch join하면 row 중복과 pagination 오류가 난다.
- 대표 endpoint에 query count budget을 둔다.

## 개념 설명

Hibernate N+1은 parent 목록 한 번 조회 뒤 각 row의 lazy association마다 추가 SELECT가 실행되어 query count가 1+N으로 늘어나는 문제다.

SQL interceptor나 datasource proxy로 request별 query count와 normalized SQL 반복 횟수를 수집하고 fetch join·entity graph·batch fetch로 원인 association을 고친다.

## 예시

```text
SELECT * FROM orders LIMIT 50;            -- 1
SELECT * FROM customer WHERE id=?; x 50   -- N
fix: fetch join customer or batch IN (...)
```

개발 데이터 N이 작으면 latency가 드러나지 않으므로 통합 테스트에서 query count 상한과 실제 page 크기를 함께 검증한다.

## 면접 답변 예시

> Hibernate N+1은 parent 목록을 읽은 뒤 lazy association마다 추가 SELECT가 나가 request query 수가 1+N으로 늘어나는 문제입니다. Datasource proxy나 SQL interceptor로 endpoint별 normalized query 반복과 count budget을 두고 실제 page 크기로 test하겠습니다. To-one은 fetch join이나 entity graph, collection pagination은 batch fetch처럼 association 특성에 맞는 방법을 선택합니다. Query 수만 줄이고 cartesian row와 memory를 늘리지 않도록 fetched row, heap과 latency를 함께 측정하고 OSIV로 view에서 늦게 발생하는 SQL도 포함합니다.

## 장점

- 성능 회귀를 query count assertion으로 막는다.

## 단점

- query count만 줄이고 과다 row를 로딩할 수 있다.

## 주의사항 / 실무 팁

- collection pagination은 batch fetch를 우선 검토한다.
