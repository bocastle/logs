# TypeScript Discriminated Union 정리

## 핵심 요약

- status를 검사한 분기 안에서 성공 데이터와 실패 오류가 자동으로 좁혀져 불필요한 non-null 단언이 사라진다.
- 판별자 값을 일반 string으로 넓히면 TypeScript가 멤버를 구분하지 못해 각 분기에서 속성 접근이 다시 불안정해진다.
- variant 생성 함수의 status는 literal로 추론되게 하고 반환 타입으로 union 멤버의 필수 필드를 확인한다.

## 개념 설명

Discriminated Union은 공통 판별자 필드로 여러 객체 형태 중 하나를 안전하게 표현하는 타입 모델이다.

분기에서 `status` 같은 literal 값을 검사하면 TypeScript가 남은 필드를 해당 variant로 좁힌다.

## 예시

```ts
type Result = { status: "ok"; data: User } | { status: "error"; message: string };
function label(result: Result) {
  return result.status === "ok" ? result.data.name : result.message;
}
```

`status` 확인 뒤에는 성공과 실패에서 접근 가능한 필드가 달라져 잘못된 참조를 줄인다.

## 면접 답변 예시

> 로딩, 성공, 실패를 배타적인 객체로 표현하면 동시에 data와 error가 존재하는 모순된 상태를 만들기 어렵다. 검증하지 않은 API 응답을 union으로 단언하면 런타임에 알려지지 않은 status가 들어와 exhaustive 분기를 우회한다. 네트워크 payload는 schema로 판별자와 variant별 필드를 검증한 뒤 애플리케이션 union으로 변환한다.

## 장점

- 새 variant를 union에 추가하면 처리하지 않은 switch가 컴파일 오류로 드러나 상태 모델 변경 범위를 찾기 쉽다.

## 단점

- status를 optional로 두거나 여러 variant가 같은 literal을 공유하면 Discriminated Union의 좁히기 기준이 무너진다.

## 주의사항 / 실무 팁

- switch 마지막에서 남은 값을 never에 대입하는 assertNever를 호출해 멤버 추가 시 누락을 실패시킨다.
