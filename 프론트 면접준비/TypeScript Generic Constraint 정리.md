# TypeScript Generic Constraint 정리

## 핵심 요약

- `T extends { id: string }` 제약은 함수 내부의 id 접근을 허용하면서 호출자의 추가 필드 타입을 보존한다.
- 제약을 `{}`로 두면 nullish만 제외할 뿐 필요한 멤버를 설명하지 않아 안전한 연산이 거의 생기지 않는다.
- 함수 본문에서 실제 읽는 속성만 작은 interface로 표현해 제네릭 제약을 최소화한다.

## 개념 설명

Generic Constraint는 제네릭 타입 인자가 가져야 할 최소 구조를 `extends`로 제한하는 방법이다.

함수 내부에서 필요한 속성을 안전하게 쓰면서 호출자는 더 구체적인 타입 정보를 보존할 수 있다.

## 예시

```ts
function byId<T extends { id: string }>(items: T[]) {
  return new Map(items.map((item) => [item.id, item]));
}
```

id가 있는 타입만 받되 반환 Map의 value에는 원래 타입 T가 유지된다.

## 면접 답변 예시

> key 제약을 `K extends keyof T`로 두면 반환 타입을 선택한 속성의 실제 value 타입과 연결할 수 있다. constraint는 기본 타입 인자가 아니므로 호출자가 타입을 생략했을 때 특정 concrete 타입으로 고정되지 않는다. 허용해야 하는 추가 필드 객체와 필수 필드가 빠진 실패 객체를 나란히 컴파일해 API 경계를 검증한다.

## 장점

- 잘못된 타입 인자는 Generic 함수 본문이 아니라 호출 위치에서 필요한 최소 구조와 함께 오류가 난다.

## 단점

- 공통 구현 편의를 위해 너무 많은 필드를 extends에 넣으면 유효한 더 작은 객체까지 API에서 배제한다.

## 주의사항 / 실무 팁

- 반환 위치에 T를 유지해 제약의 base type으로 넓어지지 않는지 expectTypeOf로 확인한다.
