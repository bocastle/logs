# 느린 백엔드 API로 인한 사용성 저하, 프론트엔드에서 내가 하는 대응들

> ⚠️ 프론트엔드라면 고민해야 하는 문제

---

## 문제

- **증상**: API 응답이 1~3초+ 지연 → “멈췄다”는 인상, 이탈, 반복 클릭으로 중복 요청 폭증
- **원인(프론트 시점)**: 과도한 동시 요청, 큰 페이로드, 비효율 쿼리 패턴, 캐시 미활용, 렌더 차단, 취소 불가 요청 등
- **제약**: 백엔드 개선이 단기간 불가할 때도 **지금 당장** 체감 품질을 올려야 함

---

## 특징(내 접근 순서)

1. **수치화 먼저**: RUM(실사용) 기준으로 API P50/P95, 타임라인, 에러율 확보 → 팀 공유 가능한 한 장 도표
2. **체감 시간 줄이기**: Skeleton/Progress/단계적 로딩/낙관적 UI로 지연을 **‘느끼지 않게’**
3. **요청 전략화**: 캐싱/프리패치/중복제거/취소/재시도/타임아웃/우선순위로 **병목을 우회**
4. **데이터 양/형식 다이어트**: 페이지네이션, 필드 선택, 압축으로 **절대량 감소**
5. **안전장치**: 회로차단기, 폴백 데이터, UX 카피로 **최악의 순간에도 망가지지 않게**
6. **백엔드와 공통 언어**: SLA 가설(예: P95 < 1.5s) + 트레이스 스냅샷으로 **정확히** 협업

---

## 장점(이 방식의 이익)

- **즉효**: 백엔드가 느려도 **오늘** UI 사용성 방어 가능
- **예측 가능**: 타임아웃/재시도/취소로 비정상 대기 제거
- **확장 용이**: 캐시/프리패치/우선순위는 기능 확대에도 재사용

---

## 단점(주의할 점)

- **낙관적 UI 롤백 비용**: 비가역 작업이면 보정 로직이 복잡
- **캐시 불일치 리스크**: stale 관리/무효화 전략 필요
- **과도한 메모화·프리패치**: 메모리·네트워크 낭비 가능 → 조건부 적용

---

## 최적화(전략 & 코드 스니펫)

### 1) 요청 취소·타임아웃·재시도(지수 백오프+지터)

```js
// js: AbortController + timeout + retry with jitter
const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

async function fetchWithControls(
  url,
  { signal, timeout = 6000, retries = 2, baseDelay = 300 } = {}
) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    const ctrl = new AbortController();
    const timer = setTimeout(() => ctrl.abort("timeout"), timeout);
    try {
      const res = await fetch(url, {
        signal: signal ?? ctrl.signal,
        keepalive: true,
        cache: "no-store",
      });
      clearTimeout(timer);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res;
    } catch (e) {
      clearTimeout(timer);
      if (e.name === "AbortError" && e.message !== "timeout") throw e; // 외부 취소면 즉시 중단
      if (attempt === retries) throw e;
      const jitter = baseDelay * 2 ** attempt + Math.random() * 200;
      await sleep(jitter);
    }
  }
}
```

### 2) 중복 요청 제거(“in-flight” 디듀플)

```js
// js: 같은 키의 요청이 진행 중이면 그 프라미스를 재사용
const inFlight = new Map();

async function dedupFetch(key, factory) {
  if (inFlight.has(key)) return inFlight.get(key);
  const p = factory().finally(() => inFlight.delete(key));
  inFlight.set(key, p);
  return p;
}

// 사용
dedupFetch(`user:${id}`, () => fetchWithControls(`/api/users/${id}`));
```

### 3) React Query로 캐시·프리패치·낙관적 UI

```js
// ts/js with @tanstack/react-query
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000, // 30s 동안 재요청 방지
      gcTime: 5 * 60_000, // 캐시 보관
      retry: 1, // 서버가 느릴 때 과도한 재시도 방지
      refetchOnWindowFocus: false,
    },
  },
});

// 프리패치(호버/뷰포트 진입 시)
function prefetchUser(id) {
  return queryClient.prefetchQuery({
    queryKey: ["user", id],
    queryFn: () => fetchWithControls(`/api/users/${id}`).then((r) => r.json()),
  });
}

// 낙관적 업데이트
function useUpdateName() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (name) =>
      fetch("/api/name", { method: "POST", body: JSON.stringify({ name }) }),
    onMutate: async (name) => {
      await qc.cancelQueries({ queryKey: ["me"] });
      const prev = qc.getQueryData(["me"]);
      qc.setQueryData(["me"], (old) => ({ ...old, name })); // 즉시 반영
      return { prev };
    },
    onError: (_e, _v, ctx) => ctx?.prev && qc.setQueryData(["me"], ctx.prev), // 롤백
    onSettled: () => qc.invalidateQueries({ queryKey: ["me"] }),
  });
}
```

### 4) 단계적 로딩: Skeleton → Placeholder → 실제 데이터

```js
// React 18 Suspense + skeleton 패턴(데이터 라이브러리와 함께 사용 권장)
function CardSkeleton() {
  return (
    <div className="animate-pulse space-y-2">
      <div className="h-4 w-2/3 bg-gray-200 rounded" />
      <div className="h-4 w-1/2 bg-gray-200 rounded" />
      <div className="h-24 w-full bg-gray-200 rounded" />
    </div>
  );
}
```

### 5) keepPreviousData로 “점프” 방지(페이지네이션 체감 개선)

```js
// React Query: 페이지 전환 시 이전 페이지 유지로 깜빡임 제거
const { data, isFetching } = useQuery({
  queryKey: ["list", page],
  queryFn: () => fetchJSON(`/api/list?page=${page}`),
  placeholderData: keepPreviousData,
});
```

### 6) 프리패치 트리거(사용자 의도 기반)

```js
// 링크에 마우스 올리면 다음 화면 데이터 미리 가져오기
<Link to={`/product/${id}`} onMouseEnter={() => prefetchProduct(id)}>
  상세보기
</Link>
```

### 7) 요청 우선순위/병렬화 제어

```js
// 낮은 우선순위 요청은 유휴 시점에
requestIdleCallback?.(() => prefetchLightData());

// 중요 데이터 먼저, 비중요는 뒤로
await Promise.all([
  fetchCritical(), // above-the-fold
  fetchNonCritical().catch(() => {}), // 실패해도 UX 유지
]);
```

### 8) 데이터 다이어트(필드 선택 & 페이징 & 압축)

- select/fields 파라미터로 필요한 필드만
- 서버/클라이언트 압축(gzip/br) 확인
- 이미지 썸네일 우선 로딩 → 원본 지연
- 페이지네이션/무한스크롤로 초기 페이로드 축소

### 9) 접근성 & UX 디테일

```js
// 접근성: 로딩 영역에 aria-busy/aria-live
<div aria-busy={isLoading} aria-live="polite">
  {isLoading ? <Skeleton /> : <Content />}
</div>
```

- 버튼은 비활성화+스피너로 중복 클릭 방지
- 친절한 카피: “저장 중…(최대 5초)” 같이 기대 관리
- 취소 버튼 제공(AbortController 연동)

### 10) 회로차단기 & 폴백

```js
// 단순 회로차단기 예시: 최근 n회 실패/타임아웃이면 단기간 폴백 사용
let failCount = 0;
const LIMIT = 3;
let openUntil = 0;

async function guardedFetch(url) {
  const now = Date.now();
  if (now < openUntil) return getFromCacheOrFallback(url);
  try {
    const res = await fetchWithControls(url, { timeout: 5000 });
    failCount = 0;
    return res;
  } catch (e) {
    failCount++;
    if (failCount >= LIMIT) {
      openUntil = now + 15_000; // 15s 오픈
    }
    return getFromCacheOrFallback(url);
  }
}
```

## 전략 요약 & 트레이드오프

| 전략                         | 효과              | 언제 쓰나                    | 트레이드오프/주의 |
| ---------------------------- | ----------------- | ---------------------------- | ----------------- |
| Skeleton/Progress            | 체감 시간 ↓       | 초기/상단 중요 영역          | 실제 속도는 동일  |
| 프리패치                     | 대기 0처럼 느끼게 | 다음 동선 예측 가능할 때     | 오예측=낭비       |
| 캐시(stale-while-revalidate) | 즉시 응답+후갱신  | 데이터 신선도 민감도 낮을 때 | 스테일 관리 필요  |
| 취소/타임아웃                | 불필요 대기 제거  | 네트워크 품질 불안정 시      | 복잡성 증가       |
| 재시도(백오프+지터)          | 일시 장애 흡수    | 간헐적 타임아웃/5xx          | 서버 과부하 주의  |
| 낙관적 UI                    | 즉시 인터랙션     | 가역/쉽게 롤백 가능 작업     | 롤백 로직 필요    |
| keepPreviousData             | 깜빡임/점프 방지  | 페이지네이션/필터 전환       | 일시 스테일       |
| 데이터 다이어트              | 절대 응답 시간 ↓  | 대용량 리스트/상세           | 서버 협의 필요    |
| 회로차단기/폴백              | 최악 상황 UX 유지 | API 불안정/간헐 장애         | 정확도 낮아질 수  |

## 결론

- 지연은 관리 대상입니다. “느리다”에서 끝내지 말고 수치화 → 우선순위화 → 사용자 체감 완화 → 요청 전략화 → 안전장치 순서로 처리합니다.
- 프론트엔드는 취소·타임아웃·캐시·프리패치·낙관적 UI만으로도 체감 품질을 크게 끌어올릴 수 있습니다.
- 마지막으로, 측정치(P95, 타임아웃율) 를 꾸준히 팀 공유해 공통 언어를 만들면, 백엔드 개선과 프론트 전략이 함께 수렴합니다.
