# API Authentication과 Authorization 분리 정리

## 핵심 요약

- 401과 403 오류 의미가 분명해진다.
- 인증 성공만 보고 자원 권한을 생략하면 IDOR 취약점이 생긴다.
- subject, action, resource, tenant를 권한 판단 입력으로 고정한다.

## 개념 설명

Authentication은 호출자가 누구인지 확인하는 과정이고, Authorization은 그 주체가 특정 자원·행동을 할 수 있는지 판단하는 과정이다.

API는 인증 미들웨어에서 subject와 tenant를 확정하고, 권한 검사는 handler나 policy engine에서 resource id와 action 기준으로 수행한다.

## 예시

```text
authn: JWT -> subject=user_7, tenant=t1
authz: can(user_7, "order.cancel", order.tenant=t1)
```

인증된 사용자라도 `order.cancel` 권한과 tenant 소유권을 따로 확인해야 수평 권한 상승을 막을 수 있다.

## 면접 답변 예시

> Authentication은 호출자의 신원을 확인하는 단계이고 authorization은 그 주체가 특정 action을 특정 resource에 수행할 수 있는지 판단하는 단계입니다. 인증 middleware에서 검증한 subject와 tenant를 request context에 넣고 handler나 policy layer에서는 `subject, action, resource, tenant`를 모두 확인하겠습니다. Token이 유효하다는 이유만으로 URL의 resource ID를 신뢰하면 IDOR가 생기므로 실제 소유권과 tenant 경계를 매 요청에서 검증합니다. 인증 정보가 없거나 유효하지 않으면 401, 인증됐지만 권한이 없으면 403을 기본으로 하되 resource 존재 노출 정책과 audit log의 민감정보를 함께 정합니다.

## 장점

- 권한 정책을 API별로 테스트하기 쉽다.

## 단점

- 권한 로직이 handler마다 흩어지면 정책이 어긋난다.

## 주의사항 / 실무 팁

- 401과 403 응답 코드를 일관되게 사용한다.
