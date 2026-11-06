# Password Reset Token 설계 정리

## 핵심 요약

- 비밀번호 분실 복구를 안전하게 제공할 수 있다.
- 긴 만료 시간은 탈취 링크의 위험을 키운다.
- 재설정 완료 후 기존 세션과 refresh token 처리 정책을 정한다.

## 개념 설명

Password Reset Token은 비밀번호 재설정 권한을 짧은 시간 동안 증명하는 일회용 비밀값이다.

토큰은 충분한 난수로 만들고 hash만 저장하며, 사용자·목적·만료·사용 여부를 묶어 한 번 사용하면 즉시 폐기한다.

## 예시

```text
token = random(32 bytes)
store sha256(token), user_id, exp=15m, used=false
consume with atomic used=true
```

DB에는 token hash만 저장해야 재설정 테이블 유출 때 바로 계정 탈취로 이어지지 않는다.

## 면접 답변 예시

> Password reset token은 충분한 entropy를 가진 짧은 수명의 일회용 bearer secret입니다. 원문은 email link에만 보내고 DB에는 hash, user·purpose·expiration과 사용 상태를 저장하겠습니다. Consume 시 hash 일치와 만료를 확인하면서 unused에서 used로 바꾸는 조건부 update를 한 transaction에서 수행해 두 요청 중 하나만 성공하게 합니다. 완료 후 기존 session과 refresh token 폐기 정책을 적용하고 reset 요청 응답은 account 존재 여부와 처리 시간을 드러내지 않게 맞춥니다.

## 장점

- 일회용 처리로 링크 재사용을 막는다.

## 단점

- 토큰 원문을 저장하면 DB 유출 때 즉시 악용된다.

## 주의사항 / 실무 팁

- 요청 응답은 계정 존재 여부를 노출하지 않게 맞춘다.
