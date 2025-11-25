# CommonJS vs ES Module(ESM) 정리

> 자바스크립트의 대표적인 모듈 시스템 **CommonJS(CJS)** 와 **ES Module(ESM)** 의 차이점, Node.js/브라우저 동작, 상호 운용, 마이그레이션 팁을 정리합니다.

---

## 1) 핵심 비교 표

| 구분             | CommonJS (CJS)                                        | ES Module (ESM)                                           |
| ---------------- | ----------------------------------------------------- | --------------------------------------------------------- |
| 도입/표준        | Node.js 디팩토 표준(역사적)                           | ECMAScript 2015 공식 표준                                 |
| 로딩 모델        | **동기** `require()` (Node 파일 시스템 기준)          | **비동기** `import`(브라우저 네이티브), Node도 ESM 로더   |
| 내보내기         | `module.exports` / `exports.foo = ...`                | `export default` / `export const foo = ...`               |
| 가져오기         | `const foo = require('./foo')`                        | `import foo, { bar } from './foo.js'`                     |
| 바인딩           | **값 복사(스냅샷에 가까움)**                          | **라이브 바인딩**(원본 변경 시 참조측도 반영)             |
| 정적 분석        | 어려움(동적 require 가능)                             | 용이(정적 `import` 문) → **트리 쉐이킹** 유리             |
| 실행 시점        | `require` 순간 즉시 평가                              | 모듈 그래프 분석 후 평가(토폴로지 기반)                   |
| 순환 참조        | 허용하나 **부분 초기화** 주의(CJS 캐시의 불완전 객체) | 허용하나 **라이브 바인딩 + TDZ** 규칙 적용                |
| 최상위 await     | 미지원                                                | **지원**(`top-level await`)                               |
| 파일 확장자/구성 | `.js` 기본, Node에서 CJS                              | Node: `package.json`의 `"type"` 또는 `.mjs`/`.cjs` 확장자 |
| 런타임           | 서버 중심(번들러 필요 시 브라우저도 가능)             | 브라우저/Node 모두 네이티브 지원(현대 환경)               |

> 주의: “CJS는 값 복사”는 개념적 요약입니다. 실제로는 CJS **모듈 캐시의 내보낸 객체 레퍼런스**를 받지만, **구조분해 결과는 스냅샷**처럼 동작할 수 있습니다. ESM은 **바인딩 자체가 라이브**입니다.

---

## 2) 문법 대비

### CommonJS

```js
// math.cjs
const TWO = 2;
function add(a, b) {
  return a + b;
}
module.exports = { TWO, add };
// 사용
const { add, TWO } = require("./math.cjs");
console.log(add(TWO, 3));
```

### ES Module

```js
// math.mjs
export const TWO = 2;
export function add(a, b) {
  return a + b;
}
export default function inc(x) {
  return x + 1;
}
// 사용
import inc, { add, TWO } from "./math.mjs";
console.log(add(TWO, inc(3)));
```

---

## 3) Node.js에서의 모듈 판별 규칙

1. **`package.json`의 `"type"`**

- `"type": "module"`: `.js` → **ESM**, `.cjs` → CJS
- `"type": "commonjs"`(기본): `.js` → **CJS**, `.mjs` → ESM

2. **확장자 우선 규칙**

- `.mjs` → 항상 ESM, `.cjs` → 항상 CJS

3. **브라우저**

- `<script type="module" src="/app.js">` → ESM
- 동적 로딩: `import('/path/mod.js')`

---

## 4) 해석/해결(Resolution) 차이

- CJS: **동적** `require(id)` 가능, 런타임 문자열 결합이 허용되어 **정적 해석 어렵고 트리 쉐이킹 비효율**.
- ESM: **정적** `import` → 번들러가 종속 그래프를 빌드/최적화.
- Node ESM: 기본적으로 **확장자 명시** 요구(`import './foo.js'`). CJS의 **확장자 생략/디렉터리 인덱스** 관례와 다릅니다(번들러는 다르게 동작 가능).

---

## 5) 바인딩/캐싱/순환 참조

- **CJS**: `module.exports` 객체를 **한 번 평가 후 캐싱**(`require.cache`). 순환 참조 시 **부분 채워진 exports**가 전달될 수 있음.
- **ESM**: 각 `export` 식별자는 **라이브 바인딩**. 순환 참조에서도 **식별자 단위로 연결**되며, **TDZ(Temporal Dead Zone)** 규칙을 준수해야 합니다.

```js
// ESM 라이브 바인딩 예
// a.mjs
export let n = 0;
export function inc() {
  n += 1;
}
// b.mjs
import { n, inc } from "./a.mjs";
console.log(n); // 0
inc();
console.log(n); // 1  ← 라이브 바인딩
```

---

## 6) `default` 와 이름 있는 내보내기

- CJS의 `module.exports = value` ≒ ESM의 `export default value`.
- 상호 변환 시 **중간 래핑**에 주의: CJS를 ESM에서 가져오면 `{ default: ... }` 형태로 래핑될 수 있습니다.

```js
// CJS
module.exports = function hello() {};
// ESM에서
import hello from "./cjs-module.cjs"; // default로 매핑됨
```

---

## 7) 런타임 차이/호환 포인트

- **`__filename`, `__dirname`, `require`**: ESM 모듈에서는 기본 제공되지 않습니다. 다음과 같이 대체합니다.

```js
// ESM에서 __dirname 유사 구현
import { fileURLToPath } from "node:url";
import { dirname } from "node:path";
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

- **동적 로딩**: ESM에서 CJS의 동적 로딩과 유사하게 `await import()` 사용.
- **Top‑Level Await**: ESM 전용. 초기화 단계에서 비동기 준비가 가능.

```js
// ESM
const cfg = await import("./config.mjs");
```

---

## 8) 성능/번들링 관점

- **ESM**은 정적 그래프 덕분에 **트리 쉐이킹**·**코드 스플리팅**·**프리로딩**에 유리.
- 브라우저 ESM은 **병렬/지연 로딩**을 활용 가능. HTTP/2/3 환경과 궁합이 좋습니다.
- Node.js는 ESM 로더의 **비동기 평가/해결 비용**이 있으나, 현대 빌드/캐시 전략으로 완화됩니다.

---

## 9) 선택 가이드

- **브라우저 타깃/풀스택 호환**: **ESM 권장**(표준, 최적화 이점).
- **기존 Node 생태(CJS 패키지 다수)**: 점진적 이전. 새 코드/엣지 레이어는 ESM, 레거시는 CJS 유지.
- **라이브러리 배포**: **듀얼 패키징** 권장 — `exports`/`main`/`module` 필드 조합으로 **CJS/ESM 동시 지원**(또는 별도 빌드 산출물).

```json
// package.json 예시(듀얼)
{
  "name": "libx",
  "type": "module",
  "main": "./dist/index.cjs", // CJS 진입
  "module": "./dist/index.esm.js", // 번들러용 ESM
  "exports": {
    ".": {
      "require": "./dist/index.cjs",
      "import": "./dist/index.esm.js"
    }
  }
}
```

---

## 10) 마이그레이션 체크리스트

- [ ] `package.json`의 `"type"` 또는 확장자 전략(`.mjs`/`.cjs`) 결정
- [ ] 경로에 **확장자 명시**(Node ESM 규칙)
- [ ] `require`, `module.exports` → `import`/`export`로 변환
- [ ] `__dirname`/`__filename` → `import.meta.url` 기반 대체
- [ ] CJS 의존 라이브러리는 **브리지 레이어** 또는 듀얼 번들 사용
- [ ] 빌드/번들러 설정(트리 쉐이킹·코드 스플리팅·타깃) 검증
- [ ] 순환 참조와 **TDZ** 관련 런타임 에러 테스트
- [ ] ESM에서 **Top‑Level Await** 사용 시 초기화 경로/에러 핸들링 설계

---

## 11) 요약

- **CommonJS**: 동기 `require`, `module.exports`, 역사적으로 Node 중심. 동적 로딩 유연하지만 정적 최적화는 제한.
- **ESM**: 표준 `import/export`, 정적 해석·트리 쉐이킹·TLA 지원, 브라우저/Node 모두 네이티브.
- 현대 프로젝트는 **ESM 기본**을 권장하되, **CJS와의 상호 운용/듀얼 배포**로 생태계 호환을 확보하는 전략이 바람직합니다.
