# ESM과 CJS 차이 정리

## 핵심 요약

- 정적 import graph로 tree shaking과 사전 오류 검사가 쉬워진다.
- CJS 패키지를 ESM default import로 읽을 때 도구별 결과가 다를 수 있다.
- package.json의 type과 exports 조건을 배포 형식에 맞춘다.

## 개념 설명

ESM은 표준 `import`와 `export`를 정적으로 연결하는 모듈 시스템이고 CJS는 Node.js에서 `require`와 module.exports를 실행 시점에 평가해 값을 주고받는 형식이다.

ESM import는 export 위치에 대한 live binding이라 원본 재할당을 관찰하지만 CJS require는 module.exports 객체를 캐시해 반환하며 해석 시점과 순환 의존 동작도 다르다.

## 예시

```js
// counter.mjs
export let count = 0;
export const increment = () => count++;

// consumer.mjs: count는 live binding
import { count, increment } from "./counter.mjs";
increment();
console.log(count); // 1
// CJS에서는 const counter = require("./counter.cjs") 형태를 쓴다.
```

ESM consumer가 export된 count의 변경을 live binding으로 관찰한다. CJS interop에서는 default와 named export 변환을 런타임과 번들러별로 확인해야 한다.

## 면접 답변 예시

> live binding으로 순환 참조의 값 갱신을 명시적으로 다룰 수 있다. 하나의 파일에서 두 형식을 섞으면 실행 환경이 모듈 타입을 오판한다. 브라우저 대상 코드는 정적 import 경로와 올바른 MIME 응답을 확인한다.

## 장점

- 브라우저와 서버에서 같은 표준 모듈 문법을 사용할 수 있다.

## 단점

- top-level await가 있으면 ESM graph 평가 순서가 지연될 수 있다.

## 주의사항 / 실무 팁

- dual package는 ESM과 CJS가 서로 다른 singleton을 만들지 검사한다.
