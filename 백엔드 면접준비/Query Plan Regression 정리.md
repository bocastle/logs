# Query Plan Regression 정리

## 핵심 요약

- 성능 저하를 query text가 아닌 plan 변화로 식별한다.
- EXPLAIN ANALYZE가 실제 query를 실행해 부하를 줄 수 있다.
- 상위 fingerprint의 plan hash를 배포 전후 저장한다.

## 개념 설명

query plan regression은 SQL이 같아도 통계·데이터 분포·index·DB version 변화 뒤 더 느린 실행 계획이 선택되는 성능 퇴행이다.

배포 전후 plan hash, EXPLAIN ANALYZE의 join order, actual rows, buffer read를 fingerprint별로 비교해 회귀 지점을 찾는다.

## 예시

```text
fingerprint=8f31 old plan hash=a12 Nested Loop p95=40ms
new plan hash=f77 Seq Scan p95=2.8s
EXPLAIN ANALYZE: estimated=10 actual=180000
```

plan 고정은 급한 완화가 될 수 있지만 stale 통계나 cardinality 오차의 원인을 해결하지 않으면 다른 bind에서 손해가 난다.

## 면접 답변 예시

> Query plan regression은 SQL text가 같아도 통계, data 분포와 index·DB version 변화로 더 느린 plan이 선택되는 현상입니다. 상위 query fingerprint의 plan hash와 latency를 배포 전후 비교하고 estimate와 actual row 차이, join order와 buffer read를 보겠습니다. `EXPLAIN ANALYZE`는 query를 실제 실행하므로 write나 무거운 query에는 안전한 환경과 옵션을 선택해야 합니다. Plan pinning은 급한 완화로 제한하고 stale 통계, index와 query 구조의 원인을 별도로 해결합니다.

## 장점

- 통계 갱신과 배포를 latency 변화에 연결한다.

## 단점

- plan hash만 같아도 runtime row 수가 달라질 수 있다.

## 주의사항 / 실무 팁

- estimate와 actual 차이가 큰 node를 먼저 본다.
