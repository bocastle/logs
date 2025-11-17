# 자바스크립트 클로저(Closure) 정리

> **클로저**는 함수가 **정의된 렉시컬 스코프(lexical scope)** 를 기억하여, **함수가 생성된 이후**에도 그 스코프의 식별자에 접근할 수 있게 해주는 메커니즘입니다. 일급 함수 + 렉시컬 스코프가 결합하여 성립합니다.

---

## 1) 핵심 개념

- **렉시컬 스코프**: 함수가 **정의된 위치**를 기준으로 상위 스코프가 결정됩니다(실행 위치가 아니라 **선언 위치**).
- **환경 레코드(환경)**: 실행 컨텍스트에 저장된 식별자(변수/함수) 테이블. 내부 함수가 상위 환경을 **참조**합니다.
- **클로저**: 내부 함수가 **상위 환경 레코드에 대한 참조**를 유지하여, 외부 함수가 종료된 뒤에도 상위 변수에 접근 가능.

```js
function outerFunction(outerVariable) {
  return function innerFunction(innerVariable) {
    console.log("Outer Variable:", outerVariable);
    console.log("Inner Variable:", innerVariable);
  };
}

const fn = outerFunction("outside");
fn("inside"); // 'outside', 'inside'
```

---

## 2) 대표 활용 패턴

### 2.1 데이터 은닉(캡슐화)

```js
function createCounter() {
  let count = 0; // 외부에서 직접 접근 불가(은닉)
  return {
    inc() {
      count += 1;
    },
    dec() {
      count -= 1;
    },
    value() {
      return count;
    },
  };
}

const c = createCounter();
c.inc();
c.inc();
console.log(c.value()); // 2
```

- `count`는 클로저로 은닉되어, 메서드를 통해서만 제어됩니다.

### 2.2 비동기 컨텍스트 보존

```js
function createLogger(name) {
  return function () {
    console.log(`Logger: ${name}`);
  };
}
const logger = createLogger("MyApp");
setTimeout(logger, 1000); // 1초 후 'Logger: MyApp'
```

- 타이머/이벤트/프로미스 콜백이 **정의 시점의 변수**를 유지합니다.

### 2.3 모듈 패턴

```js
const userModule = (function () {
  let users = [];
  function add(u) {
    users.push(u);
  }
  function all() {
    return users.slice();
  }
  return { add, all }; // 공개 API만 노출
})();

userModule.add({ id: 1 });
console.log(userModule.all());
```

- 즉시 실행 함수(IIFE)로 **프라이빗 상태**와 **공용 API**를 구분.

### 2.4 커링/부분 적용·팩토리

```js
const add = (a) => (b) => a + b;
const add10 = add(10);
console.log(add10(5)); // 15
```

- 매개변수를 **부분 고정**하여 재사용 가능한 함수를 생성.

### 2.5 메모이제이션

```js
function memoize(fn) {
  const cache = new Map();
  return function (x) {
    if (cache.has(x)) return cache.get(x);
    const v = fn(x);
    cache.set(x, v);
    return v;
  };
}

const slowSquare = (n) => {
  /* 무거운 계산 */ return n * n;
};
const fastSquare = memoize(slowSquare);
```

- `cache`를 은닉하여 **결과 재사용**.

---

## 3) 자주 겪는 함정과 대응

### 3.1 루프와 `var` 캡처 문제

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 3, 3, 3
}
```

- `var`는 **함수 스코프**라서 하나의 `i`를 공유합니다.
- 해결책
  1. **`let` 사용**(블록 스코프): 각 반복마다 새로운 바인딩
  2. IIFE로 바인딩 고정

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0,1,2
}
```

### 3.2 가변 객체 캡처

- 클로저로 **참조형**(객체/배열)을 캡처하면, **외부에서 변경** 시 내부 동작이 달라질 수 있습니다.
- 필요 시 **방어적 복제** 또는 **불변 구조**(예: 구조 분해 복사, `Object.freeze`)를 활용.

### 3.3 메모리 누수

- 장수(長壽) 콜백이 큰 객체를 **참조**하면, GC가 해제하지 못해 누수 위험.
- 해결: 이벤트 리스너/타이머/구독은 **해제(cleanup)**, 필요 데이터만 캡처.

### 3.4 this와 클로저 혼동

- 클로저는 **스코프 체인**을 위한 것이고, `this` 바인딩은 **호출 방식**에 의해 결정됩니다.
- 화살표 함수는 **렉시컬 this**를 사용하므로, 콜백에서 `this` 유지가 필요할 때 유리.

---

## 4) 실행 흐름 관점(개요)

1. 외부 함수 실행 → **환경 레코드 생성**(지역 변수 바인딩).
2. 내부 함수 정의 → **상위 환경에 대한 참조**를 **[[Environment]]** 에 저장.
3. 외부 함수 종료 후에도 내부 함수가 살아있다면, 상위 환경은 **GC 대상에서 제외**되어 유지.
4. 내부 함수 호출 시, **스코프 체인**을 따라 상위 식별자 해석.

---

## 5) 실무 팁

- **필요한 최소 정보만 캡처**: 불필요한 대형 객체 캡처 금지.
- **클린업 습관화**: 타이머/리스너/구독 해제.
- **블록 스코프 사용**: 루프에는 `let`을 기본값으로.
- **순수 함수 지향**: 테스트 용이성과 예측 가능성 강화.
- **모듈화**: 클로저로 은닉된 상태는 외부 API로만 조작하도록 설계.

---

## 6) 요약

- 클로저는 **정의 시점의 스코프**를 기억하는 함수로, **데이터 은닉**, **비동기 컨텍스트 유지**, **모듈/팩토리/메모이제이션** 등에 널리 쓰입니다.
- 루프의 `var` 캡처, 가변 참조 캡처, 클린업 누락에 주의하고, **필요 최소 캡처 + 클린업** 원칙을 지키는 것이 안전합니다.
