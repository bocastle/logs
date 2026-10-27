# TypeScript Indexed Access Type 정리

## 핵심 요약

- `ApiResponse['data']`는 원본 필드 타입을 직접 참조해 응답 모델 변경 시 중복 alias도 함께 갱신된다.
- K가 여러 key의 union이면 T[K]도 value union이 되어 특정 key와 값의 상관관계를 잃을 수 있다.
- generic index는 `K extends keyof T`로 제한하고 인자 key와 반환 T[K]가 같은 타입 매개변수를 공유하게 한다.

## 개념 설명

Indexed Access Type은 객체나 배열 타입에서 특정 key의 value 타입을 `T[K]` 형태로 가져오는 문법이다.

`keyof`와 조합하면 API 응답의 특정 필드, 배열 원소, 설정 값 타입을 중복 없이 추출한다.

## 예시

```ts
type User = ApiResponse["user"];
type Item = Cart["items"][number];
```

원본 응답 타입이 바뀌면 추출 타입도 함께 바뀌어 중복 선언을 줄인다.

## 면접 답변 예시

> `Items[number]`로 배열 원소 union을 추출해 별도 Element 타입 선언 없이 목록 계약을 재사용한다. noUncheckedIndexedAccess 없이 동적 배열 접근을 모델링하면 범위를 벗어난 실제 undefined가 반환 타입에 보이지 않는다. noUncheckedIndexedAccess를 켠 프로젝트에서는 동적 lookup의 undefined 분기를 호출부에서 명시적으로 처리한다.

## 장점

- `T[K]`를 generic getter 반환에 사용하면 선택한 key마다 정확한 value 타입을 호출자에게 돌려준다.

## 단점

- tuple에 number를 인덱스로 쓰면 위치별 타입이 하나의 union으로 합쳐져 고정 slot 의미가 사라진다.

## 주의사항 / 실무 팁

- tuple 위치가 중요하면 0이나 1 literal index를 사용하고 전체 원소 집합이 필요할 때만 number를 사용한다.
