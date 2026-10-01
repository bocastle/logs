# React Server Component 직렬화 제약 정리

## 핵심 요약

- Server Component에서 조회와 포맷팅을 끝내면 해당 라이브러리 코드와 비밀 자격 증명이 클라이언트 번들에 들어가지 않는다.
- 일반 callback을 클라이언트 prop으로 넘기면 RSC payload가 함수를 표현할 수 없어 빌드나 렌더에서 실패한다.
- Client Component가 실제로 읽는 primitive, plain data, 식별자만 골라 props로 전달하고 나머지는 서버에 남긴다.

## 개념 설명

React Server Component는 서버에서 계산한 결과를 클라이언트로 보낼 때 직렬화 가능한 props만 넘길 수 있다.

함수, class instance, 브라우저 객체처럼 전송할 수 없는 값은 Client Component 경계 안에서 만들거나 server action으로 분리한다.

## 예시

```tsx
// Server Component
return <UserCard user={{ id: user.id, name: user.name }} />;
```

plain object만 넘기면 RSC payload가 안정적으로 만들어지고 클라이언트 번들 경계도 선명해진다.

## 면접 답변 예시

> 작은 payload를 Client Component에 넘기면 서버가 렌더한 정적 UI를 유지하면서 필요한 부분만 hydrate할 수 있다. 큰 조회 결과를 그대로 경계에 넘기면 HTML 이후의 RSC payload와 클라이언트 메모리 비용이 불필요하게 커진다. production RSC 렌더 테스트에서 함수, 인스턴스, 과대 payload가 경계를 통과하지 않는지 확인한다.

## 장점

- 직렬화 가능한 props 계약은 서버 계산 결과와 브라우저 상호작용 데이터의 경계를 명시적으로 만든다.

## 단점

- class instance와 브라우저 객체를 전송하려 하면 prototype 동작이 보존되지 않고 직렬화 계약도 깨진다.

## 주의사항 / 실무 팁

- 사용자 동작은 serializable 인자와 검증을 갖춘 server action 또는 명시적 API 호출로 모델링한다.
