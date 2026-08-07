# TypeScript unknown과 any 차이 정리

## 핵심 요약

- 외부 값을 unknown으로 받으면 속성 접근 전에 검증이 강제되어 신뢰 경계가 코드에 남는다.
- any에 존재하지 않는 메서드를 호출해도 컴파일되므로 오류가 실제 사용자 경로까지 늦게 발견된다.
- fetch 응답, postMessage, storage처럼 시스템 밖에서 들어오는 값의 최초 타입을 unknown으로 선언한다.

## 개념 설명

`unknown`은 아직 검증되지 않은 값을 표현하고, `any`는 타입 검사를 사실상 끄는 탈출구다.

`unknown` 값은 타입 가드로 좁히기 전까지 속성 접근이 막히지만, `any`는 잘못된 호출도 컴파일을 통과시킨다.

## 예시

```ts
function parse(value: unknown) {
  if (typeof value === "string") return value.trim();
  return "";
}
```

외부 JSON처럼 신뢰할 수 없는 값은 `unknown`으로 받은 뒤 분기에서 안전하게 좁힌다.

## 면접 답변 예시

> any가 라이브러리 반환 타입에서 호출부 전체로 전파되는 문제를 unknown 경계에서 차단할 수 있다. catch 변수와 JSON 파싱 결과를 곧바로 Error나 도메인 객체로 간주하면 원시값 예외에서 처리 코드가 다시 실패한다. noImplicitAny와 useUnknownInCatchVariables를 켜고 불가피한 any에는 범위와 제거 조건을 주석으로 남긴다.

## 장점

- 타입 가드로 한 번 좁힌 값은 해당 블록에서 구체 타입으로 추론되어 안전성과 자동완성을 함께 얻는다.

## 단점

- unknown을 검사 없이 as User로 단언하면 표기만 바뀌고 런타임 안전성은 any를 쓴 것과 다르지 않다.

## 주의사항 / 실무 팁

- 반복되는 구조 검사는 value is T 가드나 검증 schema로 만들고 실패 이유를 호출자에게 명시적으로 돌려준다.
