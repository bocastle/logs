# 리액트 동시성 모드(Concurrent Rendering) 정리

> 요약: 리액트 18의 **동시성 기능**은 렌더링을 **중단·재개·폐기**할 수 있게 하여, **더 중요한 업데이트를 우선 처리**하고 **부드러운 상호작용**을 보장합니다. 렌더링은 백그라운드에서 진행되다가 준비가 되면 한 번에 커밋됩니다.

---

## 1) 배경과 핵심 개념

- **기존(동기) 렌더링**: 한 번 시작하면 끝까지 멈추지 않음 → 긴 작업 중 **프레임 드랍**·입력 랙 가능.
- **동시성 렌더링**: 렌더 단계가 **인터럽트 가능**(중단/취소/재시작). 우선순위를 바꿔 **사용자 입력, 애니메이션, 스크롤**을 먼저 처리.
- **렌더 vs 커밋**: 렌더는 계산(가상 트리 준비), 커밋은 실제 DOM 반영. **커밋은 항상 동기**·원자적으로 적용됩니다.

> 용어: 과거 “Concurrent Mode”라 부르던 전체 스위치가 아닌, React 18부터는 **기능 단위로 동시성**을 사용합니다(예: `startTransition`, `Suspense`).

---

## 2) 동작 원리(간략)

- **Fiber 스케줄러**가 작업을 **우선순위 레인(lanes)** 으로 분류하여 병행 관리.
- 낮은 우선순위 렌더링은 진행 중이라도 **더 높은 우선순위 이벤트**(타이핑/클릭) 발생 시 **중단**될 수 있음.
- 준비가 끝난 트리는 **커밋 시점에 한 번에 적용**되어 **UI 일관성**을 보장.

---

## 3) 주요 API 및 패턴

### 3.1 `startTransition` / `useTransition`

- **덜 중요한 업데이트**를 트랜지션으로 표시하여 **입력 지연 방지**.

```tsx
import { startTransition, useState, useTransition } from "react";

// 함수형 API
function onChange(keyword: string) {
  setInput(keyword); // 높은 우선순위(입력 반영)
  startTransition(() => {
    setFiltered(filterHugeList(keyword)); // 낮은 우선순위(무거운 연산)
  });
}

// 훅 API
const [isPending, start] = useTransition();
start(() => doLowPriorityUpdate());
// isPending으로 스피너/상태 표시
```

### 3.2 `useDeferredValue`

- 값의 업데이트를 **지연/완화**하여 고빈도 입력 중 **리렌더 폭주 억제**.

```tsx
const deferredQuery = useDeferredValue(query);
const list = useMemo(() => filterHugeList(deferredQuery), [deferredQuery]);
```

### 3.3 `Suspense` (데이터/코드 분할과 결합)

- **준비되지 않은 서브트리** 대신 `fallback` 렌더. 준비되면 자동 **치환**.
- 데이터 계층(React Query의 `suspense: true`, Relay, RSC)이나 **코드 스플리팅**과 함께 사용.

```tsx
<Suspense fallback={<Skeleton />}>
  <UserPanel /> {/* 내부에서 Promise를 던지는 데이터 로딩 */}
</Suspense>
```

### 3.4 네비게이션과 트랜지션(예: 라우팅)

- 페이지 전환을 **트랜지션**으로 분류하여 **이전 화면의 응답성**을 유지.
- Next.js App Router와 결합 시 **스트리밍 SSR/선택적 하이드레이션**과 시너지.

---

## 4) 언제 쓰면 좋을까요?

- **검색/자동완성**: 입력은 즉시 반영, 결과 필터링은 트랜지션으로 백그라운드 처리.
- **대규모 리스트/차트**: 스크롤·호버 등 상호작용을 우선, 비가시 영역 계산은 지연.
- **페이지 전환/탭 전환**: 전환 중에도 기존 화면의 입력·애니메이션 **부드러움 유지**.
- **비동기 로딩 UI**: `Suspense` 경계로 스켈레톤을 **영역 단위**로 일관 표기.

---

## 5) 주의사항과 안티패턴

1. **남용 금지**: 모든 업데이트를 트랜지션으로 감싸면 **지연·깜빡임** 증가.
2. **경계 설계**: `Suspense`를 과도하게 중첩하면 **fallback 깜빡임**·비일관 UX. 상위 레이아웃은 고정, 데이터 섹션만 경계.
3. **외부 스토어 일관성**: React 외부 상태관리(예: 커스텀 이벤트 버스)와 결합 시 **스냅샷 일관성**을 확보하도록 통합 API 사용.
4. **CPU 바운드 작업**: 무거운 동기 계산은 Web Worker로 **오프로딩**을 검토.
5. **측정 기반 적용**: TTI/INP(입력 지연), 프레임 드랍 등 **지표 확인 후** 부분 적용.

---

## 6) 실전 예시(검색 입력 최적화)

```tsx
function SearchBox({ list }: { list: string[] }) {
  const [query, setQuery] = useState("");
  const [result, setResult] = useState<string[]>([]);
  const [isPending, startTransition] = useTransition();

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // 입력은 즉시
    startTransition(() => {
      // 무거운 필터링은 낮은 우선순위
      setResult(expensiveFilter(list, value));
    });
  }

  return (
    <div>
      <input value={query} onChange={onChange} placeholder="검색..." />
      {isPending && <span className="dim">필터링 중…</span>}
      <ul>
        {result.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 7) SSR/RSC와의 연계(요점)

- **스트리밍 SSR**: 먼저 준비된 섹션부터 **점진적 전송** → 초기 체감 속도 향상.
- **선택적 하이드레이션**: 상호작용 필요한 컴포넌트부터 우선 하이드레이트.
- **Server Components**: 데이터 페칭을 서버로 옮겨 **클라이언트 부담** 감소, 클라 측은 **트랜지션**으로 상호작용 최적화.

---

## 8) 요약 체크리스트

- [ ] 입력·스크롤·애니메이션 등 **상호작용 우선**이 필요한가?
- [ ] 무거운 렌더/계산/데이터 로딩을 **트랜지션/지연**으로 돌릴 수 있는가?
- [ ] `Suspense` 경계를 **역할 단위**로 배치했는가?
- [ ] 측정 기반으로 **부분 적용**하고, 깜빡임·지연을 **모니터링**하는가?

---

## 결론

동시성 기능은 “빠르게 반응하는 UI”를 위해 **업데이트 우선순위를 제어**하고 **중단 가능한 렌더링**을 제공합니다. `startTransition`/`useDeferredValue`/`Suspense`를 상황에 맞춰 조합하면, 복잡한 화면에서도 **부드러운 입력 경험**과 **일관된 로딩 UX**를 달성할 수 있습니다.
