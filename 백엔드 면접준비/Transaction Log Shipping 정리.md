# Transaction Log Shipping 정리

## 핵심 요약

- block 단위로 전체 database를 복제한다.
- 누락된 WAL segment 하나가 이후 replay를 막는다.
- segment 연속성과 checksum을 검증한다.

## 개념 설명

transaction log shipping은 primary가 생성한 WAL 또는 transaction log segment를 standby로 지속 전달해 물리 복제와 시점 복구 기반을 만드는 방식이다.

archive 또는 streaming 전송으로 log를 보내고 standby가 순서대로 replay해 동일한 storage page 변경을 재현한다.

## 예시

```text
primary WAL segment 00000001000000A1000000FF
ship -> standby archive
standby replay -> recovery LSN advance
```

segment 전송만 성공하고 replay가 멈출 수 있으므로 shipping 지연과 apply 지연을 분리해 관찰한다.

## 면접 답변 예시

> Transaction log shipping은 primary의 WAL 같은 물리 변경 log를 standby에 순서대로 전달하고 replay하는 복제 방식입니다. Application row 의미를 해석하지 않아 PITR과 standby 구축에 같은 log를 활용할 수 있지만 보통 다른 major version이나 다른 storage 형식으로 직접 replay할 수는 없습니다. Segment 하나가 누락되면 그 뒤 recovery가 멈출 수 있어 연속성과 checksum을 검증하겠습니다. 전송 성공만 보지 않고 ship, receive와 replay 위치를 따로 관찰하며 archive 삭제 전 모든 standby와 복구 보관 요구를 확인합니다.

## 장점

- PITR과 standby 구축에 같은 log를 활용한다.

## 단점

- archive 보관량이 storage를 채울 수 있다.

## 주의사항 / 실무 팁

- ship·receive·replay LSN을 각각 모니터링한다.
