# TypeScript Exhaustive Check 정리

## 핵심 요약

- never 기반 Exhaustive Check는 union에 새 상태를 추가하는 순간 빠진 렌더링 분기를 컴파일 오류로 보여준다.
- default에서 임의 문자열을 반환하면 새 variant가 조용히 일반 처리되어 exhaustive 보장이 사라진다.
- assertNever 함수는 인자를 never로 받고 도달 시 판별자 값을 포함한 오류를 던지도록 구현한다.

## 개념 설명

Exhaustive Check는 union의 모든 variant를 처리했는지 `never`로 확인하는 패턴이다.

switch의 default에서 남은 값이 `never`가 아니면 새 variant가 추가됐는데 분기가 빠진 상태로 컴파일 오류를 낸다.

## 예시

```ts
function assertNever(value: never): never { throw new Error(String(value)); }
switch (state.kind) {
  case "idle": return null;
  case "error": return state.message;
  default: return assertNever(state);
}
```

상태 종류가 늘어날 때 렌더링 분기 누락을 컴파일러가 알려준다.

## 면접 답변 예시

> 도메인 이벤트와 reducer 전이를 같은 판별 union으로 관리할 때 변경 누락을 여러 소비 지점에서 찾는다. 외부 payload가 검증되지 않았다면 타입에 없는 판별자도 런타임에 들어와 예상하지 못한 default 경로를 탄다. API 응답은 runtime schema로 union 멤버를 확인한 뒤 exhaustive 로직에 전달해 정적 계약과 실제 값을 맞춘다.

## 장점

- 모든 case가 값을 반환하게 만들면 호출자가 undefined fallback을 별도로 처리할 필요가 없다.

## 단점

- 남은 값을 `as never`로 강제하면 assertNever가 받아도 실제 누락은 컴파일러에 드러나지 않는다.

## 주의사항 / 실무 팁

- switch의 각 case에서 즉시 return하고 마지막 줄에 assertNever를 두어 control-flow 좁히기를 활용한다.
