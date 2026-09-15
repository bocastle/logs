# TypeScript Variance 정리

## 핵심 요약

- callback 매개변수의 반공변 관계를 이해하면 더 좁은 입력만 처리하는 함수를 넓은 소비자 자리에 넣는 오류를 막는다.
- 메서드 매개변수의 이변적 호환성에 기대면 일반 함수 property와 다른 결과가 나와 안전성을 과대평가할 수 있다.
- tsconfig에서 strictFunctionTypes를 활성화하고 이벤트 handler 대입의 성공 및 실패 사례를 타입 테스트로 둔다.

## 개념 설명

Variance는 제네릭 타입의 하위 타입 관계가 타입 인자 관계를 따라가는지 설명하는 개념이다.

읽기 전용 위치는 공변적으로, 함수 인자처럼 값을 소비하는 위치는 반공변적으로 동작해 callback 타입 안전성에 영향을 준다.

## 예시

```ts
type Reader<T> = () => T;
type Writer<T> = (value: T) => void;
```

Reader는 값을 꺼내기만 하고 Writer는 값을 받기 때문에 대입 가능성 판단이 달라진다.

## 면접 답변 예시

> strictFunctionTypes가 고차 함수의 대입 가능성을 검사해 런타임에 처리 못 할 subtype 전달을 줄인다. Variance 오류를 없애려고 callback 인자를 any로 바꾸면 소비 위치의 제약이 호출 체인 전체에서 사라진다. 복잡한 라이브러리 타입에는 in과 out variance annotation을 사용하되 실제 구조와 일치하는지 컴파일러로 확인한다.

## 장점

- Producer와 Consumer 제네릭을 분리해 API가 값을 읽는지 쓰는지 타입 관계에 명확히 표현할 수 있다.

## 단점

- mutable 배열을 공변처럼 넘긴 뒤 더 넓은 값을 push하면 원래 원소 타입의 가정이 깨질 수 있다.

## 주의사항 / 실무 팁

- 읽기 API는 readonly 반환을, 쓰기 API는 입력 전용 함수를 제공해 한 제네릭에 양방향 책임을 섞지 않는다.
