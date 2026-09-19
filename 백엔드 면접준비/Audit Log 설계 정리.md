# Audit Log 설계 정리

## 핵심 요약

- 업무 레코드 수정과 관계없이 조사 이력을 유지할 수 있다.
- append-only 스트림 전송 실패가 누락으로 이어질 수 있다.
- audit 전송 실패를 outbox로 재시도하고 누락 알림을 둔다.

## 개념 설명

Audit Log 설계는 중요 상태 변경과 보안 행위를 append-only 이벤트로 남기고 무결성, 접근 제어, retention을 보장하는 저장·조회 아키텍처다.

이벤트를 일반 업무 DB와 분리한 append-only 저장소에 보내고, hash chain·WORM 정책으로 tamper-evident 특성을 만들며 데이터 분류별 retention을 적용한다.

## 예시

```text
event -> append-only audit stream
record_hash = SHA256(previous_hash + canonical_event)
archive -> WORM storage, retention=3y
verify hash chain daily
```

tamper-evident는 수정을 물리적으로 불가능하게 하는 것만을 뜻하지 않고, 변경이 있었는지 검증하고 증명할 수 있게 하는 것이다.

## 면접 답변 예시

> Audit log는 누가 언제 어떤 대상에 어떤 변경을 했는지 업무 record와 독립적으로 남기는 조사 기록입니다. 업무 transaction에서 audit 전송이 빠지지 않도록 outbox 같은 내구성 있는 경로를 쓰고 append-only 저장소의 접근 권한을 운영 log와 분리하겠습니다. Hash chain은 변경 흔적을 드러내는 장치이지 조작을 물리적으로 막는 보장은 아니므로 root hash와 검증 결과를 독립 계정에 보관합니다. 민감정보는 필요한 최소 field만 남기고 retention 만료 시 legal hold를 확인한 뒤 파기합니다.

## 장점

- hash chain과 WORM 보관으로 사후 조작 탐지 근거를 만든다.

## 단점

- hash chain 검증 키와 root hash를 같은 저장소에만 두면 조작 탐지 신뢰가 낮아진다.

## 주의사항 / 실무 팁

- tamper-evident 검증 결과와 root hash를 독립된 계정에 보관한다.
