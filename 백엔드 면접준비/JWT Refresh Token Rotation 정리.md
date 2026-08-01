# JWT Refresh Token Rotation 정리

## 핵심 요약

- Refresh token rotation은 갱신할 때마다 새 토큰을 발급하고 이전 토큰을 한 번만 사용할 수 있게 만든다.
- JWT 서명 검증만으로는 이미 사용된 토큰인지 알 수 없어 `jti` hash와 token family 상태 저장소가 필요하다.
- Family의 현재 generation 갱신은 원자적으로 처리하고 이전 토큰 재사용을 탈취 신호로 다룬다.

## 개념 설명

Refresh token을 JWT로 만들었더라도 rotation은 stateful한 보안 정책이다. 갱신마다 새 refresh token을 발급하고 기존 `jti`를 consumed로 표시해 같은 bearer token을 계속 사용할 수 없게 한다.

각 토큰에는 유일한 `jti`, family ID와 generation을 넣는다. 서버는 원문 대신 `jti` hash를 저장하고 현재 generation의 compare-and-swap에 성공한 요청만 다음 토큰을 만든다. 이미 소비된 토큰이 grace 정책 밖에서 다시 오면 family 전체를 폐기하고 재인증을 요구할 수 있다.

## 예시

```text
JWT claims: {jti: "rt_8", family: "f1", generation: 8}
store hash(jti), token family=f1, status=ACTIVE
rotate -> issue rt_9 and mark rt_8 CONSUMED
```

JWT 서명은 토큰이 발급자에게서 왔는지만 확인한다. 폐기와 재사용 여부는 알려 주지 않으므로 상태 조회를 피할 수 없다. 저장소와 application log에는 JWT 원문을 남기지 않는다.

## 면접 답변 예시

> Refresh token rotation은 갱신할 때마다 새 토큰을 발급하고 이전 토큰을 소비 상태로 바꾸는 방식입니다. JWT 서명만으로는 재사용을 알 수 없으므로 `jti` hash와 token family의 현재 generation을 서버에 저장합니다. 상태 갱신은 원자적으로 한 요청만 성공하게 하고, 소비된 토큰이 다시 보이면 grace window인지 탈취 의심인지 구분해 family 폐기를 검토합니다. JWT 원문은 저장·로그에 남기지 않고 키 회전과 세션 수명 정책을 따로 테스트합니다.

## 장점

- Token family 단위로 특정 기기의 세션과 이후 토큰을 함께 폐기할 수 있다.
- 이전 토큰 재사용을 계정 탈취 신호로 탐지할 수 있다.

## 단점

- Token family 상태 갱신이 원자적이지 않으면 여러 후속 토큰이 동시에 active가 될 수 있다.
- 매 갱신의 상태 조회와 쓰기가 session 저장소의 가용성과 지연에 의존한다.

## 주의사항 / 실무 팁

- Token family 폐기를 의심 로그인 알림과 재인증 흐름에 연결한다.
- 동시 refresh에서 한 rotation만 성공하는지와 reuse 탐지 시 access token 처리 정책을 테스트한다.
