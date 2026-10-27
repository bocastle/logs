# Database Schema Drift 정리

## 핵심 요약

- 배포 전 환경별 DDL 차이를 발견한다.
- 자동 교정 DDL은 운영 객체를 잘못 삭제할 수 있다.
- schema diff는 먼저 read-only report로 운영한다.

## 개념 설명

schema drift는 migration history가 같다고 믿는 환경들 사이에서 실제 table, column, index, constraint 정의가 달라진 상태다.

운영과 기준 schema를 catalog에서 추출해 정규화한 schema diff를 만들고 migration checksum과 수동 DDL audit를 함께 비교한다.

## 예시

```text
expected: orders.status varchar(32) NOT NULL
production: orders.status varchar(16) NULL
migration history says V42 applied -> drift detected
```

migration table만 비교하면 DBA의 수동 index 생성이나 실패 후 부분 적용을 놓치므로 actual catalog가 최종 근거다.

## 면접 답변 예시

> Schema drift는 migration history가 같아 보여도 실제 table, column, index와 constraint가 환경별로 달라진 상태입니다. Migration table만 비교하면 수동 DDL과 부분 실패를 놓치므로 database catalog snapshot을 정규화해 기준 schema와 diff하겠습니다. DB가 자동 생성한 implicit object와 승인된 예외는 allowlist로 noise를 줄이되 처음에는 read-only report로 운영합니다. 차이는 자동 삭제하지 않고 원인을 확인해 정식 migration으로 환원하며 추출 권한이 불완전하지 않은지도 검증합니다.

## 장점

- 부분 적용 migration을 복구한다.

## 단점

- DB별 implicit object가 noisy diff를 만든다.

## 주의사항 / 실무 팁

- 허용 예외와 drift를 구분하는 allowlist를 둔다.
