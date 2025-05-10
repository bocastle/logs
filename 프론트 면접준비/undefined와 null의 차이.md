# undefined와 null의 차이점에 대해서 설명해주세요

## 기본 개념

`undefined`와 `null`은 둘 다 '값이 없다'는 의미를 담고 있지만, 그 쓰임새와 의미에는 차이점이 존재합니다.

### ✅ undefined

- 자바스크립트에서 **자동으로 할당되는 값**입니다.
- 변수를 선언했지만, 아직 아무 값도 할당하지 않았을 때 할당됩니다.

```javascript
let a;
console.log(a); // undefined
```

### ✅ null

- **개발자가 의도적으로 할당하는 값**입니다.
- 특정 변수에 값이 없음을 명확하게 표현하고자 할 때 사용합니다.

```javascript
let b = null;
console.log(b); // null
```

---

## 비교

- 느슨한 비교 (`==`)에서는 `null`과 `undefined`가 같게 처리됩니다.
- 엄격한 비교 (`===`)에서는 다르게 처리됩니다.

```javascript
null == undefined; // true
null === undefined; // false
```

---

## 메모리 관리 관점

### null

- 명시적으로 메모리를 해제할 때 사용됩니다.
- 객체 참조를 끊으면 가비지 컬렉터가 메모리에서 제거할 수 있습니다.

```javascript
let bigObject = {
  /* ... */
};
bigObject = null; // 참조 해제
```

### undefined

- 자바스크립트 엔진이 자동으로 할당하며, 메모리 해제와 직접적인 관련은 없습니다.
- 값이 정의되지 않음을 나타낼 뿐입니다.

```javascript
let a;
console.log(a); // undefined
```

---

## 요약

| 구분        | undefined                    | null                                 |
| ----------- | ---------------------------- | ------------------------------------ |
| 부여 주체   | 자바스크립트 엔진            | 개발자                               |
| 의미        | 값이 아직 정의되지 않음      | 의도적으로 값이 없음을 명시          |
| 비교 연산자 | `undefined == null` → `true` | `undefined === null` → `false`       |
| 메모리 영향 | 없음 (참조 해제와 무관)      | 있음 (참조 해제를 통해 GC 유도 가능) |

---

> 🤔 **Tip:** 대용량 객체의 참조를 끊고 메모리를 정리하려면 `null`을 명시적으로 할당하세요.
