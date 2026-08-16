# TypeScript Branded Type 정리

## 핵심 요약

- UserId와 OrderId가 모두 string이어도 서로 대입되지 않아 식별자 순서를 바꾼 API 호출을 컴파일 단계에서 막는다.
- `value as UserId` 단언을 곳곳에서 허용하면 brand 생성 규칙을 우회해 보증이 이름뿐인 계약이 된다.
- parseUserId가 길이와 문자 규칙을 확인한 뒤에만 UserId를 반환하도록 생성 경로를 하나로 제한한다.

## 개념 설명

Branded Type은 같은 원시 타입이라도 검증된 값에 가상의 표식을 붙여 서로 섞이지 않게 하는 기법이다.

교차 타입에 unique symbol 속성을 더하고 생성 함수를 통해서만 brand를 부여한다.

## 예시

```ts
type UserId = string & { readonly __brand: "UserId" };
function asUserId(value: string): UserId {
  if (!/^usr_/.test(value)) throw new Error("invalid user id");
  return value as UserId;
}
```

문자열을 그대로 쓰지 않고 검증된 UserId만 받게 해 주문 id와 사용자 id 혼용을 줄인다.

## 면접 답변 예시

> 통화, 거리, 정규화된 이메일처럼 표현은 같지만 의미가 다른 값의 단위를 함수 시그니처에 보존한다. 라이브러리마다 서로 다른 unique symbol을 선언하면 같은 도메인 식별자도 패키지 경계에서 호환되지 않는다. 혼동 비용이 큰 식별자와 단위에만 brand를 적용하고 모든 문자열을 감싸는 과도한 모델은 피한다.

## 장점

- 검증 함수만 Branded Type을 만들게 하면 형식 검사를 통과한 값과 원시 입력의 경계가 드러난다.

## 단점

- JSON 직렬화 결과에는 brand 표식이 없으므로 역직렬화한 문자열을 검증 없이 branded 값으로 사용할 수 없다.

## 주의사항 / 실무 팁

- brand symbol은 외부에 노출하지 않고 타입과 안전한 constructor만 export해 임의 객체 생성을 어렵게 한다.
