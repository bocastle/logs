# 이터러블(iterable)과 이터레이터(iterator)의 차이 🧐

## 문제

자바스크립트에서 반복 가능한 객체를 다룰 때, **이터러블**과 **이터레이터**라는 두 가지 개념이 자주 등장한다. 하지만 이 둘의 역할과 차이를 명확히 이해하지 못하면 혼동하기 쉽다.

## 특징

### 이터러블 (Iterable)

- **정의**: `Symbol.iterator` 메서드를 가지고 있는 객체
- **역할**: "순회할 수 있는 능력"을 제공하는 객체
- **예시**: `Array`, `String`, `Set`, `Map`
- **조건**: `[Symbol.iterator]()`를 호출했을 때 **이터레이터 객체**를 반환해야 한다.

### 이터레이터 (Iterator)

- **정의**: `next()` 메서드를 가지는 객체
- **역할**: 실제로 값을 하나씩 꺼내는 "순회 도구"
- **반환값**: `next()` 호출 시 `{ value, done }` 형태의 객체 반환
  - `value`: 현재 반환되는 값
  - `done`: 순회가 끝났는지 여부 (`true`면 종료)

## 장점

- **이터러블**은 범용적인 "순회 가능성"을 제공하고,
- **이터레이터**는 "실제 순회 동작"을 수행한다.
- 이 두 개념이 분리되어 있어, 하나의 이터러블 객체에서 여러 이터레이터를 생성할 수도 있다.

## 단점

- 헷갈리기 쉽다. 특히 초보자는 "이터러블 객체 자체가 곧 순회되는 것"으로 오해하기 쉽다.
- 이터레이터를 직접 구현하려면 다소 번거롭다.

## 코드 예시

```js
// 이터러블
const arr = [10, 20, 30];

// 이터레이터 생성
const iterator = arr[Symbol.iterator]();

console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 20, done: false }
console.log(iterator.next()); // { value: 30, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

## 결론

- **이터러블(iterable)**: `Symbol.iterator`를 가진 "순회 가능한 객체"
- **이터레이터(iterator)**: `next()` 메서드로 실제 순회를 수행하는 "도구 객체"

즉, 이터러블은 _"순회할 수 있는 자격"_,  
이터레이터는 **"순회하면서 값 꺼내는 실제 행위"**를 담당한다고 이해하면 된다.
