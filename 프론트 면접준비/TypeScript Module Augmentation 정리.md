# TypeScript Module Augmentation 정리

## 핵심 요약

- declare module로 라이브러리 SessionData에 userId를 추가하면 미들웨어 이후 코드가 확장 필드를 안전하게 읽는다.
- Module Augmentation은 선언만 합치므로 런타임 prototype이나 객체에 필드를 실제로 추가하지 않는다.
- 대상 모듈을 먼저 import한 모듈 파일 안에서 정확한 package specifier로 augmentation을 선언한다.

## 개념 설명

Module Augmentation은 기존 모듈의 타입 선언에 프로젝트별 속성이나 overload를 추가하는 선언 병합 기법이다.

`declare module` 블록에서 라이브러리 인터페이스를 확장하지만 런타임 구현이 자동으로 생기지는 않는다.

## 예시

```ts
declare module "express-session" {
  interface SessionData {
    userId: string;
  }
}
```

세션 객체 타입에 userId를 추가해 미들웨어 이후 코드에서 안전하게 참조한다.

## 면접 답변 예시

> wrapper 타입을 별도로 복제하지 않아 upstream 인터페이스 변경이 augmentation에도 자연스럽게 반영된다. 광범위한 라이브러리 인터페이스를 전역 확장하면 확장을 예상하지 않은 테스트와 다른 패키지에도 속성이 나타난다. 실제 소비 프로젝트를 컴파일하는 fixture에서 SessionData 필드 접근과 잘못된 값 대입을 함께 검사한다.

## 장점

- 기존 패키지 import를 유지한 채 프로젝트별 plugin 옵션과 overload를 공식 타입 표면에 합칠 수 있다.

## 단점

- 확장하려는 import specifier와 declare module 문자열이 다르면 전혀 다른 모듈을 선언해 소비 타입이 바뀌지 않는다.

## 주의사항 / 실무 팁

- 런타임 확장 코드와 d.ts 계약을 같은 패키지에 두고 한쪽만 배포되는 상태를 방지한다.
