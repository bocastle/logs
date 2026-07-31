# 관리자 작업 audit log 설계 기준 정리

## 핵심 요약

- 관리자 audit log는 누가 어떤 권한으로 어느 resource에 무엇을 시도했고 결과가 어땠는지 재구성할 수 있어야 한다.
- 성공뿐 아니라 권한 거절과 실패도 남기고 actor, impersonated user와 delegated role을 구분한다.
- Before·after는 allowlist 변경 요약만 저장하고 audit 저장소 자체에 별도 접근 통제와 변조 방지를 적용한다.

## 개념 설명

관리자 작업은 일반 application log보다 높은 감사 요구가 있다. Actor와 실제 적용된 role, 대상 resource, action, 업무 사유와 성공·실패를 한 사건으로 기록해야 나중에 오남용과 실수를 조사할 수 있다.

Impersonation이나 대리 작업이 있다면 로그인한 actor와 대신 행동한 subject를 나눈다. `before/after`에는 token과 개인정보 원문을 복사하지 않고 승인된 필드의 변경 요약만 남긴다. Audit event는 일반 운영자가 수정·삭제할 수 없는 저장 경계와 보존 정책을 가져야 한다.

## 예시

```json
{"actor":"admin_7","action":"USER_ROLE_CHANGE","resource":"user_9","before/after":"VIEWER->OWNER","reason":"ticket-42","request_id":"req_1","result":"SUCCESS"}
```

`request_id`를 application log와 trace에 연결하면 왜 변경됐는지와 실행 경로를 함께 볼 수 있다. Reason은 자유 텍스트만 받기보다 ticket ID와 reason code를 함께 요구하면 형식적인 입력을 줄일 수 있다.

## 면접 답변 예시

> 관리자 audit log에는 actor와 실제 권한, 대상 resource, action, reason, 변경 요약과 결과를 구조화해 남기겠습니다. 성공한 변경뿐 아니라 권한 거절과 실패도 기록해야 반복적인 탐색과 공격 시도를 찾을 수 있습니다. Before·after는 allowlist 필드의 요약만 저장해 audit log가 새로운 개인정보 유출 경로가 되지 않게 합니다. `request_id`로 application log와 trace를 연결하고 audit 저장소의 접근과 변경 자체도 별도로 감사합니다.

## 장점

- 승인 ticket과 실제 관리자 작업을 연결해 내부 통제를 검증할 수 있다.
- 실패한 고권한 시도까지 보안 탐지와 사고 조사에 활용할 수 있다.

## 단점

- Reason을 자유 텍스트로만 받으면 형식적인 값으로 채워져 업무 근거를 확인하기 어렵다.
- Audit 저장소의 권한과 무결성이 약하면 고권한 사용자가 자신의 흔적을 바꿀 수 있다.

## 주의사항 / 실무 팁

- Before·after는 allowlist 필드의 변경 요약만 남기고 token과 민감 원문을 제외한다.
- Audit event write 실패 시 관리자 작업을 허용할지 차단할지 위험도별 정책을 명시한다.
