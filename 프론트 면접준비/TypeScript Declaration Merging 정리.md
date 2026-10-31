# TypeScript Declaration Merging 정리

## 핵심 요약

- 같은 이름의 interface를 여러 선언 파일에서 확장해 plugin이 핵심 타입에 옵션을 추가할 수 있다.
- 서로 다른 dependency가 같은 전역 interface 이름을 확장하면 예상하지 않은 필드와 충돌이 프로젝트 전체에 퍼진다.
- 전역 확장 파일에는 `export {}`를 두고 declare global 범위를 명시해 우발적인 script 선언을 피한다.

## 개념 설명

Declaration Merging은 같은 이름의 interface나 namespace 선언을 TypeScript가 하나로 합치는 규칙이다.

라이브러리 타입 확장에는 유용하지만 전역 이름 충돌이나 의도하지 않은 속성 추가를 조심해야 한다.

## 예시

```ts
interface Window { analytics?: AnalyticsClient }
window.analytics?.track("open");
```

브라우저 전역에 붙인 analytics 객체를 프로젝트 타입에서 명시적으로 다룬다.

## 면접 답변 예시

> namespace와 함수 선언을 합쳐 callable API에 정적 helper를 표현하는 기존 JavaScript 패턴을 모델링한다. 동일 property를 호환되지 않는 타입으로 재선언하면 소비 코드가 아니라 멀리 떨어진 선언 파일에서 오류가 발생한다. 런타임 script 로드 여부를 검사하는 코드와 merged interface의 compile fixture를 함께 유지한다.

## 장점

- Window의 analytics property를 선언 병합하면 브라우저 전역 접근에 구체 API 자동완성이 제공된다.

## 단점

- Declaration Merging은 JavaScript 객체를 생성하지 않으므로 analytics script가 없으면 타입과 달리 런타임 값은 undefined다.

## 주의사항 / 실무 팁

- 프로젝트 prefix가 있는 이름을 사용하고 확장 surface를 최소화해 제3자 선언과의 충돌 가능성을 낮춘다.
