# TypeScript keyof와 typeof 정리

## 핵심 요약

- `keyof typeof routes`로 RouteName을 만들면 값 객체가 허용 key의 단일 원본이 된다.
- 상수 객체를 넓은 Record<string, string>으로 먼저 주석 처리하면 keyof 결과가 string으로 넓어진다.
- 객체 literal에는 as const와 satisfies를 조합해 key 검사와 좁은 value 추론을 함께 유지한다.

## 개념 설명

`typeof`는 값에서 타입을 뽑고 `keyof`는 객체 타입의 키 union을 만든다.

상수 객체를 기준으로 허용 key와 value 타입을 파생하면 설정 이름과 코드의 중복을 줄일 수 있다.

## 예시

```ts
const routes = { home: "/", settings: "/settings" } as const;
type RouteName = keyof typeof routes;
```

`routes` 값이 바뀌면 RouteName도 함께 바뀌어 잘못된 라우트 이름을 줄인다.

## 면접 답변 예시

> 값 접근 타입을 함께 파생하면 라우트 이름에 대응하는 경로 literal 정보도 보존할 수 있다. Object.keys의 반환을 무조건 `Array<keyof T>`로 단언하면 런타임에 추가된 열거 속성을 타입이 숨길 수 있다. typed keys helper는 소유 객체에 추가 열거 속성이 없다는 조건을 문서화하고 제한된 내부 값에만 적용한다.

## 장점

- 설정 key 자동완성과 철자 검사가 실제 런타임 객체 구조를 따라가 수동 union의 누락을 줄인다.

## 단점

- 배열에 keyof를 적용하면 숫자 index뿐 아니라 length와 메서드 이름까지 포함되어 원소 key로 쓰기 어렵다.

## 주의사항 / 실무 팁

- 배열 원소 타입은 `typeof items[number]`를 사용하고 객체 key 추출과 다른 의도를 표현한다.
