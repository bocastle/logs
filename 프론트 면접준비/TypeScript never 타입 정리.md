# TypeScript never 타입 정리

## 핵심 요약

- never를 switch의 남은 값에 요구하면 union 분기가 완전히 처리됐음을 컴파일러로 증명할 수 있다.
- never를 void와 같은 의미로 사용하면 정상 반환이 없는 함수와 값을 무시하는 함수의 계약을 혼동한다.
- 예상 밖의 never가 나타나면 conditional 각 branch와 intersection property의 중간 alias를 따로 확인한다.

## 개념 설명

`never`는 도달할 수 없는 값이나 절대 반환하지 않는 흐름을 나타내는 바닥 타입이다.

union에서 모든 경우를 제거한 뒤 남은 타입이 `never`인지 확인하면 빠진 분기를 찾을 수 있다.

## 예시

```ts
function fail(message: string): never {
  throw new Error(message);
}
```

예외를 던지는 함수가 반환하지 않음을 표현해 이후 제어 흐름 분석이 정확해진다.

## 면접 답변 예시

> 분배 Conditional Type에서 조건에 맞지 않는 멤버를 never로 바꾸면 union 필터를 표현할 수 있다. 값을 as never로 단언하면 도달 불가능성 검사를 통과시키면서 실제 런타임 입력은 그대로 남는다. 반환하지 않는 함수에는 never를, 값을 반환하지 않지만 정상 종료하는 callback에는 void를 사용한다.

## 장점

- 항상 throw하거나 종료하는 함수의 반환 never가 뒤따르는 control flow에서 불가능한 타입을 제거한다.

## 단점

- 호환되지 않는 객체 타입의 교차로 property가 never가 되면 생성 가능한 값이 없는데 복잡한 alias가 원인을 숨긴다.

## 주의사항 / 실무 팁

- exhaustive helper는 never 인자를 받고 런타임 도달 시 원래 값을 포함해 throw하도록 작성한다.
