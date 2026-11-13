# TypeScript Type Guard 정리

## 핵심 요약

- Type Guard는 unknown 값을 런타임에 확인한 결과를 if 블록의 정적 좁히기와 연결한다.
- predicate가 검사하지 않은 필드까지 User라고 선언하면 TypeScript는 거짓 보증을 그대로 신뢰한다.
- guard 인자는 unknown으로 받고 객체 여부, own property, 각 필드 타입과 범위를 단계적으로 검사한다.

## 개념 설명

Type Guard는 런타임 검사 결과를 TypeScript의 타입 좁히기와 연결하는 함수나 조건문이다.

`value is T` 반환 타입을 쓰면 filter, if문 뒤에서 값의 타입이 안전하게 좁아진다.

## 예시

```ts
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value;
}
```

외부 입력을 User로 쓰기 전에 최소 필드를 확인해 잘못된 접근을 막는다.

## 면접 답변 예시

> API 경계의 검사 함수를 재사용해 여러 소비자가 같은 최소 필드와 값 범위를 기준으로 판단한다. 도메인 타입이 변경됐는데 guard 구현을 갱신하지 않으면 선언과 런타임 검사가 서로 다른 계약이 된다. 복잡한 구조는 schema에서 타입과 parser를 함께 파생해 수동 Type Guard의 변경 누락을 줄인다.

## 장점

- `value is User` predicate를 filter에 사용하면 결과 배열에서 nullable 멤버를 제거한 타입을 얻는다.

## 단점

- `'name' in value`만 확인하면 inherited property나 잘못된 value type도 통과할 수 있다.

## 주의사항 / 실무 팁

- 유효 경계값뿐 아니라 null, 배열, prototype 상속, 잘못된 중첩 필드가 false인지 테스트한다.
