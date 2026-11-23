# API Gateway 인증 위임 정리

## 핵심 요약

- 중복 토큰 검증 비용과 구현 차이를 줄인다.
- 내부 헤더를 서명 없이 믿으면 직접 호출로 위조될 수 있다.
- 서비스는 gateway에서 온 요청만 받도록 네트워크를 제한한다.

## 개념 설명

API Gateway 인증 위임은 토큰 검증, mTLS 확인, 기본 rate limit 같은 공통 인증 작업을 gateway에서 먼저 처리하는 구조다.

gateway는 검증된 identity를 서명된 내부 헤더나 mTLS 연결로 downstream에 전달하고, 서비스는 세부 authorization을 다시 판단한다.

## 예시

```text
external JWT -> gateway verify
gateway -> service: X-Principal signed, X-Tenant
service checks resource permission
```

gateway 인증은 공통 방어선이고, 서비스 권한 검사를 대체하지 않는다.

## 면접 답변 예시

> API gateway 인증 위임은 외부 token 검증과 공통 인증 정책을 gateway에서 처리하고 검증된 identity context만 service에 전달하는 구조입니다. 외부 요청의 identity header는 gateway가 먼저 제거한 뒤 서명된 header나 mTLS로 principal과 tenant를 전달하고 service network는 gateway에서만 접근 가능하게 제한하겠습니다. Downstream service는 전달된 신원을 다시 로그인시키지는 않더라도 resource별 authorization과 tenant 소유권은 직접 판단해야 합니다. Key rotation, gateway 우회 시도와 header 변조를 통합 테스트하고 인증 장애와 권한 거부를 별도 지표와 audit log로 남깁니다.

## 장점

- 외부 인증 정책을 한 곳에서 회전할 수 있다.

## 단점

- gateway 장애가 전체 인증 장애가 된다.

## 주의사항 / 실무 팁

- 전달 헤더는 서명하거나 mTLS로 보호한다.
