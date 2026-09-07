# TypeScript Assertion Function 정리

## 핵심 요약

- Assertion Function을 통과한 unknown payload는 이후 블록에서 구체 타입으로 좁혀져 반복 검사가 줄어든다.
- 실제 검사보다 넓은 타입을 asserts로 선언하면 컴파일러가 존재하지 않는 필드를 안전하다고 믿게 된다.
- assertion 본문에서 타입이 요구하는 각 필드와 값 범위를 런타임 조건으로 빠짐없이 확인한다.

## 개념 설명

Assertion Function은 함수가 반환되면 인자가 특정 타입이라고 컴파일러에 알려주는 검증 함수다.

`asserts value is T` 시그니처를 사용하고 실패 시 예외를 던져 이후 코드에서 타입을 좁힌다.

## 예시

```ts
type User = { name: string };

function assertUser(value: unknown): asserts value is User {
  if (
    typeof value !== "object" || value === null ||
    !("name" in value) || typeof value.name !== "string"
  ) {
    throw new Error("invalid user");
  }
}
assertUser(payload);
payload.name;
```

검증 함수 호출 뒤 payload를 User로 다룰 수 있어 외부 입력 처리 코드가 단순해진다.

## 면접 답변 예시

> `asserts value is User` 시그니처가 검증 뒤의 control flow를 설명해 호출부의 단언을 제거한다. 검증한 객체가 이후 외부 코드에서 mutation되면 assertion 당시의 조건이 더는 유지되지 않을 수 있다. 외부 입력이 복잡하면 수동 asserts보다 schema parser를 사용하고 검증된 immutable 도메인 값으로 복사한다.

## 장점

- 도메인 불변식과 실패 시 throw 동작을 한 함수에 묶어 여러 진입점에서 동일한 검증을 재사용한다.

## 단점

- 모든 실패를 일반 Error로 던지면 입력 오류와 서버 불변식 위반을 상위 계층에서 구분하기 어렵다.

## 주의사항 / 실무 팁

- 실패에는 전용 오류 타입과 필드 경로를 담아 API 계층이 적절한 상태 코드와 메시지로 변환하게 한다.
