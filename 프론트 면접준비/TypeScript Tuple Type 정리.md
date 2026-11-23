# TypeScript Tuple Type 정리

## 핵심 요약

- Tuple은 `[x: number, y: number]`처럼 길이와 각 위치 의미를 number[]보다 정확히 표현한다.
- label 없는 여러 string slot은 타입이 맞아도 name과 email 순서를 바꾼 실수를 잡지 못한다.
- 외부에 공개하는 tuple에는 labeled element를 사용하고 변경하지 않을 반환값은 readonly로 선언한다.

## 개념 설명

Tuple Type은 위치와 길이가 정해진 배열 형태를 표현하는 타입이다.

각 위치의 타입이 다를 수 있어 좌표, hook 반환값, parser 결과처럼 순서가 계약인 값에 쓴다.

## 예시

```ts
type Point = readonly [x: number, y: number];
const point: Point = [10, 20];
```

두 숫자의 의미를 위치 이름으로 남겨 일반 number[]보다 계약이 분명하다.

## 면접 답변 예시

> variadic tuple을 사용해 함수 parameter 앞뒤를 보존하면서 middleware 인자를 조합할 수 있다. optional과 rest element를 과도하게 섞으면 각 index의 undefined 및 union 추론이 읽기 어려워진다. 정확한 길이, slot별 타입, 잘못된 순서의 실패 사례를 expectTypeOf와 ts-expect-error로 검증한다.

## 장점

- Hook 반환값을 고정 slot으로 선언하면 구조 분해한 state와 setter가 서로 다른 타입으로 추론된다.

## 단점

- mutable tuple에 push를 허용하면 정해진 길이라는 도메인 의미와 실제 배열 내용이 달라질 수 있다.

## 주의사항 / 실무 팁

- 필드가 늘거나 선택적으로 바뀔 가능성이 크면 위치 계약보다 이름 있는 object 반환을 선택한다.
