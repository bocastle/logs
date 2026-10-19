# React Suspense 데이터 캐시 정리

## 핵심 요약

- 같은 cache key가 동일한 Promise를 재사용하면 여러 소비자가 중복 fetch 없이 한 Suspense 결과를 기다린다.
- 인증 사용자가 다른데 전역 key를 공유하면 한 사용자의 응답이 다른 세션에 반환되는 데이터 누출이 생긴다.
- cache key에 resource id뿐 아니라 권한, locale, 필터처럼 결과를 바꾸는 모든 입력을 포함한다.

## 개념 설명

React Suspense 데이터 캐시는 같은 요청의 Promise와 결과를 재사용해 렌더링 중 데이터 읽기를 안정화하는 계층이다.

캐시 key가 같으면 pending Promise를 공유하고, invalidation이나 revalidate 시점에 새 데이터를 읽도록 경계를 갱신한다.

## 예시

```tsx
const user = userResource.read(userId);
return <UserProfile user={user} />;
```

동일 userId 요청을 여러 컴포넌트가 읽어도 중복 fetch와 waterfall을 줄일 수 있다.

## 면접 답변 예시

> 서버 요청 범위 cache는 한 렌더 트리의 중복 조회를 합치면서 사용자 간 데이터는 격리한다. key와 entry를 제거하지 않는 브라우저 cache는 탐색할수록 메모리가 계속 증가한다. 서버 cache는 request 또는 사용자 scope로 제한하고 클라이언트 cache에는 TTL과 최대 크기 정책을 설정한다.

## 장점

- 데이터와 pending 상태가 resource entry에 함께 있어 렌더마다 새 요청을 시작하는 일을 막는다.

## 단점

- reject된 Promise를 무기한 보관하면 재시도 렌더도 같은 오류를 즉시 던져 Error Boundary에서 빠져나오지 못한다.

## 주의사항 / 실무 팁

- mutation 성공 시 관련 entry를 invalidate하고 오류 재시도에는 새 Promise를 만들 수 있는 명시적 경로를 둔다.
