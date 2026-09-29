# TypeScript Nominal Typing 흉내내기 정리

## 핵심 요약

- Nominal marker를 붙이면 구조가 같은 OrderId와 ProductId도 선언된 도메인 이름에 따라 구분된다.
- 호출자가 이중 단언으로 brand를 만들 수 있게 두면 nominal typing의 생성 경계가 쉽게 우회된다.
- 프로젝트 전체가 공유하는 unique symbol 선언을 한 모듈에 두고 각 nominal alias가 이를 일관되게 참조하게 한다.

## 개념 설명

TypeScript는 구조적 타입 시스템이지만 brand나 private field를 이용해 nominal typing처럼 구분할 수 있다.

같은 구조의 값이라도 가상의 marker를 달면 서로 대입되지 않아 식별자나 단위 혼용을 줄인다.

## 예시

```ts
type OrderId = string & { readonly __brand: "OrderId" };
type ProductId = string & { readonly __brand: "ProductId" };
```

둘 다 string이어도 OrderId와 ProductId를 구분해 잘못된 API 호출을 막는다.

## 면접 답변 예시

> opaque handle을 함수 반환으로만 만들게 하면 유효한 생성 절차를 통과한 값만 후속 API에 전달된다. 간단한 데이터 전달 객체까지 모두 명목화하면 구조적 호환이 주는 테스트 대역과 재사용 장점을 잃는다. 서로 바뀌면 실제 장애가 나는 식별자와 단위에 성공 및 대입 실패 타입 테스트를 집중한다.

## 장점

- private field를 가진 class는 우연히 같은 public 멤버를 가진 외부 객체와 호환되지 않아 라이브러리 경계를 보호한다.

## 단점

- 직렬화된 값은 marker를 잃으므로 네트워크 왕복 뒤에도 같은 nominal 값이라고 자동으로 가정할 수 없다.

## 주의사항 / 실무 팁

- 검증, 정규화, 생성 책임을 가진 factory만 opaque 타입을 반환하고 raw 값 추출 함수는 의도를 드러내게 이름 짓는다.
