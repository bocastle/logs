# TanStack Query를 사용하는 이유 정리

> 요약: TanStack Query는 **서버 상태(Server State)**를 위한 표준화된 도구로, 캐싱·동기화·오류/로딩 상태·재요청·무효화·옵티미스틱 업데이트를 **선언적**으로 다루게 하여 네트워크 코드의 복잡도를 크게 줄입니다.

---

## 1) 서버 상태란 무엇인가

- **서버에서 생성·진실을 보유**하며, 클라이언트는 **읽기/요청**을 통해 접근하고 **직접 수정할 수 없는** 데이터.
- **비동기**·**오래됨(staleness)**·**공유됨**의 특성 때문에 일반 클라이언트 상태와 달리 별도 전략이 필요합니다.

---

## 2) TanStack Query를 쓰는 핵심 이유

### 2.1 캐싱과 신선도 제어

- **staleTime**(신선 유지 기간)·**gcTime**(미사용 시 캐시 수명)으로 네트워크 재요청을 줄이고 **즉시 응답** UX 제공.
- **백그라운드 재검증(Background Refetch)**으로 화면 끊김 없이 최신성 확보.

### 2.2 선언적 비동기 상태 관리

- `useQuery()`로 **로딩/성공/오류** 상태를 일관 처리, `enabled`, `select`, `placeholderData`, `initialData` 등으로 흐름 제어.
- `useMutation()`으로 **쓰기/갱신** 로직을 분리하고 성공·실패·정리 콜백(`onSuccess/onError/onSettled`)을 구조화.

### 2.3 자동 무효화와 동기화

- **Query Key**를 기준으로 `invalidateQueries`/`refetchQueries`로 **부분 갱신**. 의존도가 높은 화면에서도 **일관성** 유지.
- **윈도우 재포커스/네트워크 재연결** 시 자동 동기화(옵션으로 제어).

### 2.4 UX 향상 기능

- **옵티미스틱 업데이트** + 실패 시 **롤백**으로 체감 지연 감소.
- **페이지네이션/무한 스크롤**을 위한 표준 패턴(`getNextPageParam`, `useInfiniteQuery`).
- **프리패칭/사전 로드**로 전환 지연 최소화.

### 2.5 생태계·운영 편의

- **Devtools**로 쿼리 상태/캐시/키를 시각화.
- **SSR/SSG/Hydration**(Next.js 등) 통합 지원.
- **Retry/지수 백오프/취소**(AbortController) 등 네트워크 레질리언스 내장.

---

## 3) 단점/한계와 주의점

1. **캐싱 전략의 복잡성**

   - 부적절한 `staleTime/gcTime`은 **오래된 데이터 노출** 또는 **불필요한 재요청**을 유발.
   - 해결: 데이터 신선도 요구사항에 맞춰 **리소스별 기본값을 문서화**하고, 민감 리소스엔 **짧은 staleTime + 가시적 리프레시** 적용.

2. **학습 곡선**

   - 쿼리 키 설계, 무효화 범위, 옵티미스틱 업데이트, SSR 하이드레이션 등 개념이 많음.
   - 해결: **쿼리 키 규칙**(문자열 네임스페이스 + 파라미터 튜플)과 **폴더·훅 컨벤션**을 팀 표준으로 정립.

3. **클라이언트 상태와의 경계**

   - **서버 상태**와 **클라이언트 UI/폼 상태**를 혼용하면 복잡해짐.
   - 해결: UI 상태는 **React state/Zustand/Redux**, 서버 상태는 **Query**로 **역할 분리**.

4. **전역 캐시로 인한 결합**
   - 무분별한 키 공유/무효화가 **예상외 전파**를 유발.
   - 해결: **스코프 분리**(QueryClient per boundary) 또는 키 네임스페이스 준수.

---

## 4) 언제 특히 효과적인가

- **읽기 빈도 높고 변경 주기가 예측되는 리소스**(목록·상세·프로필·설정 등).
- **오프라인/불안정 네트워크**(캐시 히트로 체감 속도 개선).
- **탭 전환·라우팅 많은 SPA**(프리패치/백그라운드 동기화 효과 큼).
- **협업 대시보드/콘솔**처럼 **동기화 요구**가 큰 화면.

---

## 5) 베스트 프랙티스

### 5.1 쿼리 키 설계

```ts
// 규칙 예시: ['resource', 'scope', paramsObject]
const userKey = (id: string) => ["user", "detail", { id }];
```

- **리소스명 + 스코프 + 파라미터**를 튜플로. 문자열 결합 대신 **객체 파라미터**로 가독성↑, 부분 무효화 용이.

### 5.2 신선도 전략

- 갱신 빈도 낮음: `staleTime` 길게(분~시간), `refetchOnWindowFocus: false`.
- 빈번 갱신: 짧은 `staleTime`, 포커스/재연결 리패치 활성화.
- 비즈니스 중요 데이터: **수동 리프레시 버튼** 병행.

### 5.3 옵티미스틱 업데이트 패턴

```ts
const mutation = useMutation(updateTodo, {
  onMutate: async (variables) => {
    await queryClient.cancelQueries(todoKey.list());
    const previous = queryClient.getQueryData(todoKey.list());
    queryClient.setQueryData(todoKey.list(), optimisticUpdate(previous, variables));
    return { previous };
  },
  onError: (_err, _vars, ctx) => {
    if (ctx?.previous) queryClient.setQueryData(todoKey.list(), ctx.previous);
  },
  onSettled: () => queryClient.invalidateQueries(todoKey.list());
});
```

- **낙관적 반영 → 실패 시 롤백 → 정합성 보정** 순서.

### 5.4 선택적 구독과 셀렉터

```ts
useQuery(userKey(id), fetchUser, {
  select: (d) => ({ name: d.name, role: d.role }), // 필요한 필드만
});
```

- **`select`로 파생 데이터**를 만들어 **리렌더 범위**를 축소.

### 5.5 무한 스크롤 & 페이지네이션

- `useInfiniteQuery` + `getNextPageParam`
- **키에 페이지 파라미터 포함** 또는 커서 기반으로 **불변 리스트 병합** 유지.

### 5.6 SSR/Hydration

- 서버에서 `dehydrate(queryClient)` → 클라이언트 `Hydrate`로 불일치/깜빡임 최소화.

---

## 6) SWR 등 대안과 비교(간단)

- **TanStack Query**: 쿼리/뮤테이션 구분, **무효화·옵티미스틱 업데이트·Infinite Query** 등 기능 풍부, 대규모 앱에 적합.
- **SWR**: API 단순, 파일 크기 작음. 라이트한 페칭/캐싱 중심 앱에 적합.

---

## 7) 결론

- TanStack Query는 **서버 상태의 난제(캐싱·동기화·에러/로딩·갱신)**를 표준 패턴으로 해결하여 **생산성과 UX**를 모두 끌어올립니다.
- 다만 **키 설계·신선도 전략·역할 분리**를 팀 규칙으로 정립해 **과도한 결합**과 **오래된 데이터 노출**을 방지하시기 바랍니다.
