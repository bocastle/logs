# TypeScript Utility Types 정리

## 핵심 요약

- Pick으로 화면이 읽는 필드만 파생하면 원본 모델의 필드 이름 변경이 소비 타입에도 함께 반영된다.
- Partial은 중첩 객체 내부까지 optional로 만들지 않으므로 deep patch로 오해하면 유효한 요청을 잘못 모델링한다.
- 파생 결과에는 UserSummary나 UpdateUserInput처럼 용도를 드러내는 alias를 붙여 오류 메시지를 읽기 쉽게 만든다.

## 개념 설명

Utility Types는 `Pick`, `Omit`, `Partial`처럼 기존 타입을 조합해 새 타입을 만드는 표준 도구다.

mapped type과 conditional type을 기반으로 필드를 선택, 제외, 선택값화해 API별 계약을 줄여 쓴다.

## 예시

```ts
type UserPatch = Partial<Pick<User, "name" | "avatarUrl">>;
type PublicUser = Omit<User, "passwordHash">;
```

원본 User에서 수정 요청과 공개 응답 타입을 파생해 필드 중복을 줄인다.

## 면접 답변 예시

> 표준 Utility Types를 사용하면 팀이 자체 mapped type의 세부 규칙을 다시 학습할 필요가 줄어든다. 여러 Utility Types를 길게 중첩하면 최종 필수 필드와 readonly 여부를 리뷰에서 파악하기 어려워진다. 네트워크 DTO는 도메인 타입에서 무조건 파생하지 말고 버전과 노출 정책이 다르면 명시적 타입으로 분리한다.

## 장점

- Omit과 Partial을 조합해 생성, 수정, 공개 응답처럼 서로 다른 API 표면을 짧게 표현할 수 있다.

## 단점

- Omit<User, 'password'> 타입만 선언하고 실제 객체에서 필드를 제거하지 않으면 비밀 값은 런타임 응답에 그대로 남는다.

## 주의사항 / 실무 팁

- 필수, 선택, 금지 필드가 기대대로인지 expectTypeOf와 실패 예제를 사용해 공개 계약을 고정한다.
