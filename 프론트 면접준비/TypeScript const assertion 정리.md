# TypeScript const assertion 정리

## 핵심 요약

- as const는 action 객체의 type을 string이 아닌 정확한 literal로 보존해 reducer 좁히기를 가능하게 한다.
- const assertion은 객체를 런타임에 freeze하지 않으므로 다른 mutable alias를 통한 변경까지 막지 못한다.
- literal 표현식 가까이에 as const를 적용하고 넓은 변수에 뒤늦게 붙여 타입 정보를 복구하려 하지 않는다.

## 개념 설명

`as const`는 값의 literal 타입을 보존하고 객체와 배열을 readonly로 추론하게 한다.

설정 객체나 action type 목록에서 넓은 string 대신 정확한 literal union을 만들 때 유용하다.

## 예시

```ts
const actions = ["save", "cancel"] as const;
type Action = (typeof actions)[number];
```

배열 값에서 Action union을 만들면 허용되는 액션 이름이 값과 타입에서 함께 관리된다.

## 면접 답변 예시

> 설정 값의 구체 문자열이 남아 key별 반환 타입이나 경로 자동완성에 활용된다. 모든 값을 지나치게 좁히면 일반 string을 받는 업데이트 함수와 readonly 배열 API 사이에 불필요한 마찰이 생긴다. 상수 값에서 파생한 union을 공개해 런타임 목록과 별도로 같은 문자열을 다시 선언하지 않는다.

## 장점

- 상수 배열을 readonly tuple로 추론해 `typeof values[number]`에서 허용 값 union을 바로 만들 수 있다.

## 단점

- 외부 입력을 `as const`로 단언해도 값이 허용 schema를 만족하는지 검사하는 효과는 없다.

## 주의사항 / 실무 팁

- 객체 형태 검증도 필요하면 `as const satisfies Contract` 조합으로 literal 보존과 필드 검사를 함께 수행한다.
