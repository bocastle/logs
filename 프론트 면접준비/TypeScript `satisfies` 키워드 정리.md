# TypeScript `satisfies` 키워드 정리

## 핵심 요약

- `satisfies`는 **값의 구체적인 타입 추론은 유지**하면서, 그 값이 **특정 타입 조건을 만족하는지 검사**할 때 사용합니다.
- 타입을 강제로 “덮어씌우는” 타입 어서션(as)과 달리, **검사만 하고 추론 정보는 최대한 보존**하는 쪽에 가깝습니다.
- 특히 유니온 타입/레코드 타입에서, “정해진 키를 만족해야 하지만 값 타입은 더 구체적으로 추론되길 원할 때” 유용합니다.

## `satisfies`란?

`satisfies` 키워드는 “이 값이 **이 타입을 만족한다**”를 **검사**하되, 값 자체의 **추론 결과는 가능한 그대로 유지**하도록 도와줍니다.

- `Record<K, V>`처럼 “키/값 형태를 강제”하고 싶을 때
- 하지만 각 프로퍼티의 값은 더 구체적으로(리터럴/튜플/구체 타입) 추론되길 원할 때  
  자주 사용됩니다.

## 예시: `Record<Color, string | RGB>`에서의 타입 추론 차이

### `Record<Color, string | RGB>`로 직접 타입을 지정한 경우

아래처럼 `palette`에 타입을 직접 붙이면, 각 프로퍼티 접근 결과가 `string | RGB`로 넓게 추론될 수 있습니다.

```ts
type Color = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];

const palette: Record<Color, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
};

// ⚠️ 오류 발생: string | RGB 에는 toUpperCase가 없음
const greenNormalized = palette.green.toUpperCase();
```

### `satisfies`를 사용한 경우

`satisfies`로 “이 객체가 `Record<Color, string | RGB>`를 만족한다”를 검사하면서도, 각 프로퍼티의 구체 타입 추론을 더 잘 살릴 수 있습니다.

```ts
type Color = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];

const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Record<Color, string | RGB>;

// ✅ 정상 동작: palette.green 이 string으로 추론되어 toUpperCase 가능
const greenNormalized = palette.green.toUpperCase();
```

## 장점

- “타입 조건 충족 여부”를 검증하면서도, 값의 **구체적인 추론 타입을 보존**해 사용성이 좋아집니다.
- 키 누락/오타 같은 문제를 `Record<Color, ...>` 검사로 **컴파일 타임에 잡아낼 수** 있습니다.

## 단점

- 원문에서는 별도의 단점이 직접적으로 언급되지는 않았습니다.

## 주의사항 / 실무 팁

- `satisfies`는 **타입을 바꾸는 것(캐스팅)** 이 아니라, **타입 조건을 만족하는지 검사**하는 용도에 가깝습니다.
- “객체 형태는 특정 타입을 따라야 하지만, 각 값의 타입 추론은 최대한 유지하고 싶다”면 `satisfies`를 우선 고려하세요.
- 반대로, 정말로 타입을 강제로 바꿔야 하는 상황(의도적으로 좁히기/확장하기)이라면 `as`(타입 어서션)를 쓰는 쪽이 더 맞을 수 있습니다.
