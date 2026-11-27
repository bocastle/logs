# TypeScript Function Overload 정리

## 핵심 요약

- overload는 입력 literal이나 객체 형태에 따라 달라지는 반환 타입을 각 호출 시그니처에서 정확히 보여준다.
- 구현 시그니처는 외부 호출에 보이지 않으므로 구현이 unknown을 받아도 선언되지 않은 조합은 호출할 수 없다.
- 가장 구체적인 overload를 위에 두고 구현은 type guard로 모든 공개 조합을 빠짐없이 처리한다.

## 개념 설명

Function Overload는 하나의 구현 함수에 여러 호출 시그니처를 제공해 입력에 따른 반환 타입을 표현하는 방식이다.

공개 overload는 구체적으로 적고 구현 시그니처는 모든 경우를 처리할 수 있게 넓게 둔다.

## 예시

```ts
function parse(value: string): Date;
function parse(value: number): Date;
function parse(value: string | number) {
  return new Date(value);
}
```

호출자는 문자열과 숫자 입력을 모두 Date 반환으로 안전하게 사용할 수 있다.

## 면접 답변 예시

> 기존 호출을 유지하면서 새 입력 형식을 별도 signature로 추가해 점진적인 API 전환을 지원한다. signature 목록과 구현 분기가 따로 변경되면 타입상 허용된 호출이 런타임에서 처리되지 않을 수 있다. 각 signature의 반환 추론과 섞이면 안 되는 인자 조합을 타입 테스트로 고정한다.

## 장점

- 에디터가 허용 호출 조합을 개별 문서로 표시해 하나의 넓은 union signature보다 사용법을 찾기 쉽다.

## 단점

- 넓은 overload를 먼저 두면 더 구체적인 signature가 가려져 반환 추론이 의도보다 넓어진다.

## 주의사항 / 실무 팁

- 입력과 반환의 상관관계가 없다면 overload 대신 union parameter 하나로 API를 단순화한다.
