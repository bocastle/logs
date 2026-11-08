# Browser SameSite Cookie 정리

## 핵심 요약

- CSRF 공격면을 기본적으로 줄일 수 있다.
- OAuth redirect나 외부 결제 복귀 흐름에서 쿠키가 빠질 수 있다.
- 로그인, OAuth, 결제 복귀를 실제 cross-site 흐름으로 테스트한다.

## 개념 설명

SameSite Cookie는 브라우저가 cross-site 요청에 쿠키를 보낼지 제한해 CSRF와 추적 위험을 줄이는 쿠키 속성이다.

`SameSite=Lax`는 일반적으로 cross-site top-level의 안전한 navigation에는 쿠키를 보낼 수 있지만 subrequest와 unsafe method에는 제한한다. `None`은 cross-site 전송 의도를 나타내며 `Secure`와 함께 써야 한다.

## 예시

```http
Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Lax; Path=/
Set-Cookie: embed=def; HttpOnly; Secure; SameSite=None; Path=/
```

일반 세션은 Lax로 두고, 제3자 iframe처럼 cross-site가 필요한 쿠키만 None과 Secure를 함께 검토한다.

## 면접 답변 예시

> SameSite는 cross-site request에서 cookie를 보낼 조건을 제한해 CSRF 공격면을 줄이는 속성입니다. `Lax`는 top-level의 안전한 navigation 같은 일부 흐름을 허용하므로 이것만으로 모든 CSRF를 막는다고 보지 않고 token과 Origin 검증을 함께 사용하겠습니다. Cross-site iframe이 꼭 필요한 cookie만 `None; Secure`로 분리하고 일반 session 범위를 넓히지 않습니다. Domain이 아니라 schemeful site 기준을 이해하고 login, OAuth와 결제 복귀를 실제 browser redirect로 test합니다.

## 장점

- 쿠키의 cross-site 사용 의도를 명확히 표현한다.

## 단점

- SameSite=None에 Secure가 없으면 브라우저가 거부한다.

## 주의사항 / 실무 팁

- None 쿠키는 필요한 범위로만 분리한다.
