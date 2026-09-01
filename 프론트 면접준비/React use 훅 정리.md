# React use 훅 정리

## 핵심 요약

- use로 Promise를 읽으면 데이터 대기 상태가 가장 가까운 Suspense 경계의 fallback과 직접 연결된다.
- 렌더마다 새 Promise를 만들어 use에 넘기면 캐시가 안정되지 않아 반복 suspend와 경고가 발생한다.
- Promise는 서버, framework cache, 모듈 범위 resource처럼 렌더 밖의 안정적인 소유자가 생성하게 한다.

## 개념 설명

React `use` 훅은 Promise나 context 같은 값을 렌더링 중 읽고 준비되지 않은 Promise는 Suspense 경계로 넘긴다.

값이 pending이면 가까운 Suspense fallback이 표시되고, reject되면 Error Boundary가 처리한다.

## 예시

```tsx
function UserName({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  return <strong>{user.name}</strong>;
}
```

데이터 읽기와 대기 UI가 Suspense 경계로 연결되므로 fallback 위치 설계가 중요하다.

## 면접 답변 예시

> 서버에서 시작한 비동기 값을 클라이언트 경계로 전달하면 동일한 스트리밍 흐름에서 결과를 소비할 수 있다. 클라이언트 컴포넌트 자체를 async 함수로 바꾸면 지원 범위를 벗어나 번들러나 프레임워크에서 실패할 수 있다. 이미 보이는 콘텐츠를 새 Promise로 갱신할 때 transition을 사용해 불필요한 fallback 재노출을 줄인다.

## 장점

- context를 조건 분기에서 읽을 수 있어 일반 Hook과 달리 실제로 필요한 렌더 경로에서 값을 선택할 수 있다.

## 단점

- use 호출 주변의 try/catch로 reject를 처리하려 하면 지원되는 Error Boundary 전파 모델과 어긋난다.

## 주의사항 / 실무 팁

- pending에는 Suspense, reject에는 Error Boundary가 각각 보이도록 경계와 재시도 동작을 함께 설계한다.
