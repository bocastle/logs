
# ES6(ECMAScript 2015) 정리

> 핵심: **가독성·모듈성·비동기 처리·컬렉션 모델**을 근본적으로 개선. `let/const`, 화살표 함수, 클래스, 모듈, 템플릿 리터럴, 구조 분해, rest/spread, `Promise`, `Map/Set`, `Symbol`, 이터러블/제너레이터 등 도입.

---

## 1) 변수와 스코프
- **`let` / `const`**: 블록 스코프, 재선언 불가, TDZ로 선언 전 접근 불가.
- **`var`와 차이**: 함수 스코프, 선언 호이스팅(초기값 `undefined`), 의도치 않은 전역/중복 선언 위험.
- 권장: 기본 `const`, 변경 필요 시 `let`, `var` 지양.

```js
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
```

---

## 2) 함수 문법
- **화살표 함수**: 간결한 표기, **자체 `this`/`arguments` 없음**(상위 스코프 캡처).
- **디폴트 파라미터**:
  ```js
  const greet = (name = "guest") => `hi ${name}`;
  ```
- **rest 파라미터 / spread 연산자**:
  ```js
  const sum = (...nums) => nums.reduce((a,b) => a+b, 0);
  const arr = [1,2]; const more = [...arr, 3];
  ```

---

## 3) 클래스(Class) 문법
- 프로토타입 기반을 **클래스 문법**으로 표준화. `constructor`, `extends`, `super` 지원.
- 정적 메서드/게터·세터 등 가독성 향상, 상속 표현 간결.

```js
class Animal { speak(){ return "..." } }
class Dog extends Animal { speak(){ return "woof" } }
```

---

## 4) 템플릿 리터럴
- 백틱(`) 문자열, **보간** `${expr}`, **멀티라인** 지원.
```js
const user = "kim"; const msg = `hello, ${user}
line2`;
```

---

## 5) 구조 분해 할당(Destructuring)
- 객체/배열을 패턴으로 분해해 변수 바인딩.
```js
const { id, name: displayName } = userObj;
const [first, , third = 0] = arr;
```

---

## 6) 모듈(Modules)
- **정적 `import`/`export`**로 의존성 명시·정적 분석·트리 쉐이킹에 유리.
```js
// math.js
export const add = (a,b)=>a+b;
export default function sub(a,b){ return a-b; }

// app.js
import sub, { add } from "./math.js";
```
- 브라우저: `<script type="module">` 또는 번들러(Vite/Webpack/Rollup) 사용.
- Node.js: ESM(`.mjs` 또는 `package.json`의 `"type":"module"`)과 CJS 호환 주의.

---

## 7) 컬렉션과 새로운 타입
- **`Map` / `Set` / `WeakMap` / `WeakSet`**: 키 타입 제약 해소, 순회 보장, 서브시스템에 유용.
- **`Symbol`**: 고유 키 생성(충돌 방지), 내부 프로토콜(예: `Symbol.iterator`)에 사용.

```js
const m = new Map([[{id:1},"v1"]]);
const s = new Set([1,2,2,3]); // {1,2,3}
```

---

## 8) 이터러블/제너레이터
- **이터러블 프로토콜**(`Symbol.iterator`)과 **`for...of`** 순회.
- **제너레이터 함수**(`function*`, `yield`)로 지연 시퀀스·커스텀 이터레이터 손쉽게 구현.

```js
function* range(n){ for(let i=0;i<n;i++) yield i; }
for (const v of range(3)) {} // 0,1,2
```

---

## 9) 비동기: Promise (및 이후 async/await)
- **`Promise`**: 상태(대기→이행/거부), 체이닝(`then/catch/finally`), 에러 전파 일원화.
- ES2017의 **`async/await`**는 Promise의 문법적 설탕(ES6 외 연관 신기술).
```js
fetch(url).then(res=>res.json()).catch(console.error);
```

---

## 10) 객체 리터럴 개선
- **축약 표기**: `{ x, y }`, **계산된 프로퍼티**: `{ [key]: value }`,
  **메서드 축약**: `{ draw(){...} }` 등으로 선언 간결.

```js
const key = "score"; const obj = { key, [key]: 100, log(){ console.log(this[key]); } };
```

---

## 11) Reflect / Proxy (메타프로그래밍)
- **`Proxy`**: 객체 연산을 가로채 커스텀 동작(검증/로깅/가상화 등).
- **`Reflect`**: 내부 연산에 대한 일관 API 제공.

```js
const p = new Proxy({}, { set(t,k,v){ if(k==="age" && v<0) throw Error(); t[k]=v; return true; } });
```

---

## 12) 기타 주요 추가
- **`for...of`** (이터러블 순회), **`Number`, `Math` 확장**, **`String` 메서드 추가**,
  **`Promise.all/race`**, **모듈 로더의 표준화 기초** 등.

---

## 13) 마이그레이션과 호환성
- **트랜스파일러**: Babel/TypeScript로 ES5 타깃 번들 생성.
- **폴리필**: core-js/regenerator-runtime 등으로 런타임 기능 보강.
- 구형 브라우저 지원 범위를 **browserslist**로 명시해 빌드 체인에 반영.

---

## 14) ES6 이후 문법과의 관계
- ES2016+(ES7~)에서 **지속적으로 추가**: `async/await`, `includes`, `optional chaining(?.)`, `nullish coalescing(??)` 등.
- 실무에서는 “ES6”를 **현대 JS 전반**의 약칭처럼 쓰기도 하나, **정확히는 2015 스펙**임에 유의.

---

## 15) ES6 이전 문법도 알아야 하는 이유
- **레거시 코드 유지보수**, 점진적 마이그레이션, **트랜스파일 결과 이해**,
  호이스팅/프로토타입 등 **언어 핵심 개념**은 ES5 문맥으로도 파악 필요.

---

## 베스트 프랙티스 요약
- 기본 `const`, 변경 시 `let`, `var` 사용 지양.
- 함수는 **화살표**(콜백·클로저) + **선언식**(모듈 유틸) 혼용.
- 컬렉션은 **`Map/Set`** 우선 고려(키/중복/성능 관점).
- 모듈은 **ESM 표준** 채택, 번들러/트랜스파일 설정 일관화.
- Promise 에러 흐름을 통일하고, 필요 시 `async/await`(ES2017)로 가독성 개선.
