# TypeScript Conditional Types 정리

## 핵심 요약

- Conditional Types로 응답 variant에 맞는 data 타입을 계산하면 수동 overload 수를 줄일 수 있다.
- naked type parameter에 조건을 적용하면 union이 분배되어 전체 union을 한 번 검사하려던 결과와 달라진다.
- 분배가 필요 없으면 `[T] extends [U]`처럼 tuple로 감싸고 의도를 보여주는 별도 alias를 만든다.

## 개념 설명

Conditional Types는 `T extends U ? X : Y` 형태로 타입 조건에 따라 결과 타입을 고르는 문법이다.

union에 적용하면 각 멤버에 분배되어 API 응답이나 함수 반환 타입을 정밀하게 표현할 수 있다.

## 예시

```ts
type ApiData<T> = T extends { ok: true; data: infer D } ? D : never;
type UserData = ApiData<{ ok: true; data: User }>;
```

성공 응답에서 data 타입만 뽑아 후속 로직의 입력 타입으로 재사용한다.

## 면접 답변 예시

> union 각 멤버에 조건을 적용해 특정 속성을 가진 타입만 선택하거나 변환하는 필터를 표현한다. 재귀 조건과 큰 union을 중첩하면 타입 인스턴스화 깊이 오류와 에디터 지연이 발생한다. Awaited나 ReturnType처럼 표준 Utility Type이 같은 연산을 제공하면 검증된 내장 정의를 우선 사용한다.

## 장점

- infer와 결합해 Promise, 함수, 배열의 내부 타입을 추출하는 재사용 가능한 타입 연산을 만든다.

## 단점

- any가 조건 입력에 들어오면 양쪽 결과가 섞일 수 있고 never는 분배 과정에서 사라져 예상 타입을 숨긴다.

## 주의사항 / 실무 팁

- 단일 타입, union, never, any를 각각 넣은 expectTypeOf 사례로 조건의 경계 결과를 기록한다.
