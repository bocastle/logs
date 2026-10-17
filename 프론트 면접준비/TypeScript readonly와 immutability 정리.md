# TypeScript readonly와 immutability 정리

## 핵심 요약

- readonly property는 reducer가 입력 state를 직접 대입 변경하는 실수를 컴파일 단계에서 막는다.
- readonly는 타입 검사에만 존재하므로 JavaScript 소비자나 mutable alias가 같은 객체를 바꿀 수 있다.
- 공개 함수는 readonly 입력을 받고 변경이 필요할 때 spread나 toSorted로 새 컬렉션을 반환한다.

## 개념 설명

`readonly`는 TypeScript에서 해당 참조를 통해 값을 수정하지 못하게 하는 타입 수준 제약이다.

런타임 객체를 얼리는 것은 아니므로 깊은 불변성이나 외부 mutation 방지는 별도 설계가 필요하다.

## 예시

```ts
type State = { readonly items: readonly Item[] };
function add(state: State, item: Item): State {
  return { items: [...state.items, item] };
}
```

기존 배열을 바꾸지 않고 새 배열을 만들어 reducer와 React 변경 감지가 쉬워진다.

## 면접 답변 예시

> immutability를 지키는 업데이트는 React가 참조 변화로 수정 범위를 감지하고 memoized selector가 캐시를 갱신하기 쉽게 한다. 단언으로 readonly를 제거해 in-place sort를 수행하면 다른 참조가 관찰하는 배열 순서도 예고 없이 변한다. mutation을 시도해 실패해야 하는 타입 테스트와 이전 객체가 보존되는 런타임 reducer 테스트를 함께 둔다.

## 장점

- ReadonlyArray를 함수 인자로 받으면 정렬이나 push 없이 새 결과를 반환한다는 소비 계약이 드러난다.

## 단점

- 바깥 property만 readonly여도 중첩 객체 타입이 mutable이면 내부 필드는 계속 수정된다.

## 주의사항 / 실무 팁

- 깊은 불변성이 도메인 규칙이면 재귀 타입뿐 아니라 Object.freeze 또는 immutable 자료구조를 런타임에 적용한다.
