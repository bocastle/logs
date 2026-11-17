# TypeScript satisfies와 as const 비교 정리

## 핵심 요약

- satisfies는 객체가 목표 계약을 충족하는지 검사하면서 key와 value의 구체 정보를 변수에 남긴다.
- satisfies만 사용하면 객체가 자동으로 readonly가 된다고 오해해 후속 mutation 가능성을 놓칠 수 있다.
- 필요한 것이 shape 검사인지 literal 및 readonly 보존인지 먼저 구분해 satisfies와 as const를 선택한다.

## 개념 설명

`as const`는 값을 좁고 readonly로 만들고, `satisfies`는 값의 구체 타입을 보존하면서 대상 타입을 만족하는지 검사한다.

설정 객체에서는 `satisfies`로 필수 키를 검증하고 literal 추론은 유지하는 조합이 유용하다.

## 예시

```ts
const routes = {
  home: "/",
  user: "/users/:id",
} as const satisfies Record<string, string>;
```

경로 값은 literal로 남고 전체 객체가 문자열 map 계약을 만족하는지도 확인한다.

## 면접 답변 예시

> `as const satisfies Routes`를 조합하면 경로 literal 보존과 필수 route 검사를 한 선언에서 수행한다. 두 연산을 무조건 함께 쓰면 수정 가능한 설정까지 지나치게 좁고 readonly인 타입으로 굳어진다. property mutation 가능 여부, 누락 key 실패, value literal 추론을 각각 작은 타입 테스트로 기록한다.

## 장점

- as const는 문자열을 literal로 좁히고 배열과 property를 readonly로 만들어 값 집합 파생에 적합하다.

## 단점

- as const만 붙이면 필수 key 누락과 잘못된 value 형태를 비교할 대상 타입이 없어 검증되지 않는다.

## 주의사항 / 실무 팁

- 상수 테이블에는 `as const satisfies Contract` 순서를 사용하고 파생 union이 기대한 literal인지 확인한다.
