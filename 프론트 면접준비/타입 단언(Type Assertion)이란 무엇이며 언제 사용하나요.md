# 타입 단언(Type Assertion)이란 무엇이며 언제 사용하나요?

타입 단언(Type Assertion)은 **컴파일러에게 특정 값의 타입을 개발자가 더 정확히 알고 있다고 “단언”하는 문법**입니다.  
TypeScript가 추론한 타입보다 개발자가 의도한 타입이 더 구체적일 때 `as`(또는 `<T>` 형태)를 사용해 **추론 결과를 덮어씁니다.**

> 타입 단언은 **타입 변환(casting)** 이 아닙니다. 런타임 값이 바뀌는 것이 아니라 **컴파일 단계에서 타입만 바뀝니다.**

## 언제 사용하나요?

### 1) TypeScript가 타입을 넓게 잡을 수밖에 없는 API를 사용할 때

대표적으로 DOM API는 `null` 가능성과 다양한 엘리먼트 타입을 고려해 반환 타입이 넓게 잡혀 있습니다.

```ts
const element = document.getElementById("myElement") as HTMLDivElement;
element.style.backgroundColor = "blue";
```

- `getElementById`의 반환 타입은 보통 `HTMLElement | null`
- 하지만 개발자가 해당 요소가 `HTMLDivElement`임을 확신한다면 단언으로 좁힐 수 있습니다.

### 2) 외부 라이브러리/레거시 코드가 타입 정보를 정확히 제공하지 못할 때

- `any`로 넘어오는 값
- 제네릭이 적절히 지정되지 않아 `unknown` 또는 넓은 유니온으로 오는 값
- 런타임 보장은 있는데 타입 선언(typings)이 부족한 경우

### 3) 런타임에서는 안전하지만, 정적 분석이 그 사실을 증명하기 어려울 때

예: 특정 조건에서만 호출되며 그 조건을 통해 실제로는 `null`이 아닌데, TypeScript가 그 흐름을 추적하지 못하는 경우

## 타입 단언 사용 시 주의할 점

- 타입 단언은 **컴파일러의 타입 검사를 우회**합니다.
- 따라서 실제 런타임 값이 단언한 타입과 다르면 **런타임 에러**로 이어질 수 있습니다.
- 가능한 경우 **타입 단언 대신 타입 가드/내로잉을 우선**하는 것이 안전합니다.

## 더 안전하게 쓰기 위한 원칙

### 원칙 1) 타입 단언보다 타입 내로잉(narrowing)을 우선하기

```ts
const element = document.getElementById("myElement");
if (element instanceof HTMLDivElement) {
  element.style.backgroundColor = "blue";
}
```

### 원칙 2) 필요하다면 type predicate(타입 서술자) 기반 타입 가드를 사용하기

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

const pet: Fish | Bird =
  Math.random() > 0.5
    ? { swim: () => console.log("swim") }
    : { fly: () => console.log("fly") };

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

### 원칙 3) 단언은 “최소 범위”에서만 사용하기

객체 전체를 단언하기보다 **필요한 지점/필드만** 단언해 위험 범위를 줄입니다.

```ts
const element = document.getElementById("myElement");
if (element) {
  (element as HTMLDivElement).style.backgroundColor = "blue";
}
```

### 원칙 4) 정말 확신이 없다면 `as unknown as T` 같은 이중 단언은 피하기

이중 단언은 사실상 타입 시스템을 거의 무력화하므로, 가능한 한 **런타임 검증(스키마 검증, zod 등)** 으로 대체하는 편이 안전합니다.

## 정리

- **타입 단언**: “나는 이 값의 타입을 더 잘 알아”라고 컴파일러에게 알려 **추론 타입을 덮어쓰는 것**
- **사용 시점**: TS가 타입을 넓게 잡는 API/외부 데이터/정적 분석 한계 상황
- **주의**: 타입 검사를 우회하므로 런타임 오류 위험이 있음 → **내로잉/타입 가드 우선**, 단언은 **최소 범위**로
