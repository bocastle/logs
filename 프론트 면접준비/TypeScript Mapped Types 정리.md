# TypeScript Mapped Types 정리

## 핵심 요약

- `[K in keyof T]`로 폼 오류 타입을 만들면 원본 필드가 추가될 때 오류 key 집합도 자동으로 따라간다.
- Mapped Types의 optional 변환은 중첩 객체 안쪽까지 재귀 적용되지 않아 deep partial로 착각하기 쉽다.
- 키 매개변수는 `K extends keyof T`로 제한해 존재하지 않는 속성이 변환 대상에 들어오지 않게 한다.

## 개념 설명

Mapped Types는 키 집합을 순회하며 각 속성의 optional, readonly, value type을 변환하는 문법이다.

`keyof`와 `in`을 사용해 원본 타입의 필드 구조를 보존하면서 폼 draft나 API patch 타입을 만든다.

## 예시

```ts
type FieldErrors<T> = { [K in keyof T]?: string };
type ProfileErrors = FieldErrors<ProfileForm>;
```

폼 필드 이름과 오류 객체 키가 함께 변해 누락된 오류 매핑을 줄인다.

## 면접 답변 예시

> key remapping을 사용하면 getter 이름이나 이벤트 이름을 원본 속성에서 규칙적으로 파생할 수 있다. union 객체에 keyof를 바로 적용하면 공통 key만 남는 등 각 멤버를 순회할 때 기대와 다른 필드 집합이 나올 수 있다. 추가, 제거, remap된 key와 value 타입을 타입 테스트로 고정해 원본 모델 변경의 영향을 확인한다.

## 장점

- readonly와 optional modifier를 일괄 추가하거나 제거해 저장 모델과 편집 모델의 차이를 표현한다.

## 단점

- string index signature를 순회하면 구체 key 정보가 사라져 오타도 허용되는 넓은 결과가 만들어진다.

## 주의사항 / 실무 팁

- `-?`, `+readonly` 같은 modifier 연산을 alias 이름에 드러내고 입력과 출력 예제를 나란히 둔다.
