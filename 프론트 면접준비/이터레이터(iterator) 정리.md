# 이터레이터(iterator) 정리

## 문제

이터러블과 자주 같이 등장하는 개념이지만, **이터레이터** 자체에 대한 명확한 이해가 부족하면 둘의 차이를 구분하기 어렵다.

## 특징

- **정의**: `next()` 메서드를 가진 객체
- **역할**: 순차적으로 값을 하나씩 꺼내며, 현재 상태를 기억하는 "순회 도구"
- **반환 규약**: `next()` 호출 시 반드시 `{ value, done }` 형태의 객체를 반환
  - `value`: 현재 반환되는 값
  - `done`: 순회가 끝났는지 여부 (`true`면 더 이상 값 없음)
- 이터레이터는 **상태(stateful)**를 가진다 → 어디까지 순회했는지 추적 가능

## 장점

- 반복 동작을 세밀하게 제어할 수 있다.
- 무한 시퀀스 같은 특별한 순회 로직을 구현할 수 있다.
- for...of, 스프레드 문법 같은 자바스크립트 반복 구문과 호환된다.

## 단점

- 직접 구현하면 코드가 장황해질 수 있다.
- 배열이나 기본 이터러블 자료구조를 사용할 때는 잘 드러나지 않아 개념을 이해하기 어렵다.

## 코드 예시

```js
// 간단한 이터레이터 직접 구현
function createIterator(array) {
  let index = 0;
  return {
    next: () => {
      return index < array.length
        ? { value: array[index++], done: false }
        : { value: undefined, done: true };
    },
  };
}

const iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

## 결론

- **이터러블(iterable)**은 "순회 가능한 객체"이고,
- **이터레이터(iterator)**는 "실제 순회를 수행하는 객체"다.

이터러블이 *"순회 자격"*을 의미한다면,  
이터레이터는 그 자격을 이용해 **"실제로 하나씩 값을 꺼내는 행위"**를 담당한다고 이해하면 된다.
