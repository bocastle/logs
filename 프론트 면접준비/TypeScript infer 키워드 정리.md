# TypeScript infer 키워드 정리

## 핵심 요약

- infer로 Promise 내부 값을 추출하면 비동기 함수 반환 구조가 바뀌어도 소비 타입이 함께 갱신된다.
- union 입력의 분배를 고려하지 않으면 infer 결과가 멤버별 union이 되어 단일 공통 타입이라는 예상과 달라진다.
- conditional 왼쪽에서 먼저 감싼 형태를 제한하고 true branch 안에서 필요한 부분만 infer한다.

## 개념 설명

`infer`는 conditional type 안에서 추론된 부분 타입에 이름을 붙여 재사용하는 키워드다.

Promise 내부 값, 함수 반환 타입, 배열 원소 타입처럼 감싼 타입의 일부를 뽑을 때 사용한다.

## 예시

```ts
type AwaitedData<T> = T extends Promise<infer R> ? R : T;
type User = AwaitedData<Promise<UserDto>>;
```

Promise로 감싼 응답에서 실제 데이터 타입을 뽑아 후속 타입에 연결한다.

## 면접 답변 예시

> generic wrapper가 보존한 구체 타입을 꺼내 API adapter와 hook 결과에 다시 연결할 수 있다. 조건의 제약이 너무 약하면 매칭되지 않아야 할 타입도 infer되고 결과가 unknown이나 넓은 구조로 퍼진다. Parameters, ReturnType, Awaited 같은 내장 타입으로 충분한 경우에는 자체 infer 정의를 추가하지 않는다.

## 장점

- 함수 parameter와 return 일부에 이름을 붙여 반복되는 conditional pattern을 작은 helper type으로 캡슐화한다.

## 단점

- overload 함수에서 반환 타입을 추론하면 보통 마지막 구현 시그니처가 선택되어 개별 호출 결과를 잃는다.

## 주의사항 / 실무 팁

- 분배 여부가 중요한 helper는 union 입력과 tuple로 감싼 입력의 예상 결과를 각각 테스트한다.
