# TypeScript `satisfies` 키워드 정리

## ✅ 개념

`satisfies` 키워드는 **값이 특정 타입 조건을 충족하는지 검사하면서도, 값 자체의 구체적인 타입 정보를 보존**하도록 도와줍니다.

즉,

- `as` : 강제로 타입 단언 → 타입이 좁혀지지 않고 더 넓어질 수 있음
- `satisfies` : 타입 조건을 만족하는지 확인하면서도 원래의 타입 정보를 유지

---

## 📌 기본 예제

```ts
type Color = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number];

const palette: Record<Color, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
};

// ⚠️ 오류 발생
// palette.green 은 string | RGB 로 추론됨
const greenNormalized = palette.green.toUpperCase();
```

위 코드에서는 `palette.green`이 `string | RGB`로 추론되므로  
`.toUpperCase()`를 사용할 수 없어 에러가 발생합니다.

---

## 📌 `satisfies` 적용 예제

```ts
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Record<Color, string | RGB>;

// ✅ 정상 동작
const greenNormalized = palette.green.toUpperCase();
```

- `palette.green`이 `string`으로 정확히 추론됨
- 따라서 `.toUpperCase()`를 안전하게 호출 가능

---

## 🚀 언제 사용해야 할까?

`satisfies`는 다음 상황에서 유용합니다.

1. **유니온 타입을 다룰 때**

   - 타입이 `string | number`와 같이 넓게 추론되는 것을 막고 싶을 때

2. **객체 리터럴 타입 검증**

   - 특정 타입(`Record<Color, string | RGB>`)을 만족해야 하지만,
   - 동시에 각 속성의 **구체적인 타입 정보**를 잃고 싶지 않을 때

3. **확장 가능한 설정 객체 작성**
   - 설정 객체에 필수 필드는 보장하면서도,  
     추가 필드를 허용해야 하는 경우

---

## 📌 추가 예시: 확장 가능한 설정 객체

```ts
type Config = {
  url: string;
  method: "GET" | "POST";
};

const apiConfig = {
  url: "/users",
  method: "GET",
  cache: true, // 추가 필드
} satisfies Config;
```

- `apiConfig`는 `Config` 타입을 만족해야 함을 보장
- 동시에 `cache` 같은 추가 필드도 허용
- `method`는 `"GET"`으로 정확히 추론됨
