# Database WAL Archiving 정리

## 핵심 요약

- 원하는 시점 직전까지 transaction을 복구한다.
- archive 실패가 WAL disk 고갈로 primary를 멈출 수 있다.
- archive_command failure와 oldest unarchived segment를 알림화한다.

## 개념 설명

WAL archiving은 PostgreSQL이 완료된 WAL segment를 별도 durable storage에 복사해 PITR과 standby 복구에 사용하는 기능이다.

archive_command가 segment를 중복 안전하게 저장하고 base backup 이후 연속 WAL을 recovery target까지 재생하면 point-in-time recovery가 가능하다.

## 예시

```text
archive_command='upload-if-absent %p wal/%f'
base backup + continuous WAL archive
PITR recovery_target_time='2026-12-01 09:30:00+09'
```

archive_command 성공 코드는 실제 durable 저장과 checksum 검증을 의미해야 하며 누락 segment 하나가 이후 PITR을 막는다.

## 면접 답변 예시

> PostgreSQL WAL archiving은 완료된 WAL segment를 primary와 분리된 durable storage에 보관해 base backup 이후 원하는 시점까지 복구하는 방식입니다. `archive_command`의 성공은 단순 upload 요청이 아니라 같은 segment를 중복 안전하게 저장하고 읽을 수 있음을 의미해야 합니다. Segment 하나가 빠져도 PITR chain이 끊기므로 oldest unarchived WAL, archive failure와 primary disk를 강하게 관찰하겠습니다. Checksum과 encryption key를 포함해 base backup부터 target time까지 실제 restore drill을 반복합니다.

## 장점

- primary 장애와 별개 storage에 변경 로그를 보존한다.

## 단점

- segment 누락은 PITR chain을 끊는다.

## 주의사항 / 실무 팁

- segment checksum과 복원 가능성을 정기 검증한다.
