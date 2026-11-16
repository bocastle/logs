# Database Online DDL 정리

## 핵심 요약

- Online DDL은 테이블 변경 중에도 읽기·쓰기를 최대한 유지하는 실행 방식이지, 잠금이 전혀 없다는 뜻은 아니다.
- 작업 시작과 종료 시점의 metadata lock, 복제 지연, 추가 디스크 사용량을 함께 봐야 한다.
- 대용량 테이블에서는 취소 비용과 롤백 절차를 먼저 준비한 뒤 작은 환경에서 예상 시간을 측정한다.

## 개념 설명

DB 엔진과 변경 종류에 따라 instant, in-place, table copy 방식이 달라진다. 온라인 옵션을 지정해도 짧은 metadata lock이 필요할 수 있고, 오래 열린 트랜잭션이 있으면 시작 단계부터 대기할 수 있다.

변경 중에는 redo·WAL 증가, replica lag, 임시 디스크 사용량을 관찰해야 한다. 문제가 생겼을 때 DDL을 취소할지, 새 스키마를 되돌리는 후속 변경을 할지 롤백 기준도 미리 정한다.

## 예시

```sql
SET SESSION lock_wait_timeout = 5;
ALTER TABLE orders
  ADD COLUMN source varchar(20) NULL,
  ALGORITHM=INSTANT,
  LOCK=DEFAULT;
```

```text
중단 조건: metadata lock 대기 > 5초 또는 replica lag > 30초
롤백: 애플리케이션의 새 컬럼 사용 중지 -> 호환 배포 확인 -> 후속 DDL
```

옵션을 적는 것만으로 보장되지 않으므로 운영 DB 버전과 실제 변경 종류가 `INSTANT`를 지원하는지 사전에 확인해야 한다.

## 면접 답변 예시

> Online DDL은 서비스 트래픽을 최대한 유지하며 schema를 바꾸는 방법이지만 lock이 전혀 없다는 뜻은 아닙니다. DB version과 변경 종류에 따라 instant, in-place나 table copy가 달라지므로 실제 실행 알고리즘을 먼저 확인하겠습니다. 오래 열린 transaction과 metadata lock을 정리하고 작업 중에는 lock wait, replica lag와 disk 증가량을 관찰합니다. 중단 기준, application 호환 배포와 rollback 순서를 미리 정한 뒤 운영과 같은 크기의 환경에서 예상 시간을 측정합니다.

## 장점

- 긴 테이블 변경 중에도 서비스 중단 시간을 줄일 수 있다.
- expand-contract 배포와 결합하면 애플리케이션 호환성을 유지하기 좋다.
- 명시적인 중단 기준으로 운영 위험을 관리할 수 있다.

## 단점

- 변경 종류와 DB 버전에 따라 온라인 실행이 지원되지 않을 수 있다.
- 복제 지연과 임시 디스크 사용량이 크게 늘 수 있다.
- 취소나 롤백 자체도 오래 걸릴 수 있다.

## 주의사항 / 실무 팁

- 오래 열린 트랜잭션과 metadata lock 대기 세션을 먼저 확인한다.
- 작업 중 replica lag, WAL·redo 증가량, 디스크 여유를 관찰한다.
- NULL 허용 컬럼 추가처럼 호환 가능한 단계부터 적용한다.
