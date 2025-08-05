## ✅ infer 키워드란?

TypeScript의 `infer` 키워드는 **조건부 타입**에서 특정 타입을 추론할 때 사용됩니다.  
`extends`와 함께 조건부 타입 내에서만 사용할 수 있으며,  
어떤 타입에서 일부분을 추출하거나 해체할 때 유용합니다.

### 예제 1: 함수 반환 타입 추출

```ts
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Result1 = GetReturnType<() => string>; // string
type Result2 = GetReturnType<(x: number) => boolean>; // boolean
type Result3 = GetReturnType<string>; // never (함수가 아니므로)
```

🔍 설명:  
`T`가 함수 타입인 경우, 그 **반환 타입**을 `R`로 추론하고 결과로 사용합니다.  
`T`가 함수 타입이 아니면 `never`을 반환합니다.

---

### 예제 2: 배열 요소 타입 추출

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;

type A = ElementType<number[]>; // number
type B = ElementType<string[]>; // string
type C = ElementType<boolean>; // boolean (배열이 아니므로 원본 타입 반환)
```

---

## 📌 infer는 반드시 조건부 타입 내부에서만 사용해야 합니다.

```ts
// ❌ 아래는 문법 오류
// type Invalid = infer T;
```

---

## ✅ extends 키워드 용도 정리

### 1. 제네릭 타입 제한

```ts
type Example<T extends string> = T;

type Test1 = Example<"hello">; // 정상
// type Test2 = Example<123>; // 오류 발생 ❌
```

### 2. 조건부 타입에서의 분기 처리

```ts
type Check<T> = T extends string ? "문자열" : "다른 타입";

type Example1 = Check<"hello">; // '문자열'
type Example2 = Check<42>; // '다른 타입'
```
