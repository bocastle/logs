# useEffect 기반 로딩 상태 관리 vs React Suspense 비교 정리

> 요약: **useEffect + loading state**는 컴포넌트 내부에서 **명령형으로** 로딩을 관리합니다. **Suspense**는 **선언적 경계(boundary)** 를 두고 준비되지 않은 컴포넌트 대신 `fallback` UI를 자동으로 표시합니다. 규모가 커질수록 Suspense는 로딩 UI 조율을 단순화하지만, **Promise 기반 데이터 소스**와 **경계 설계**가 필요합니다.

---

## 1) 기존 방식: `useEffect` + `isLoading`

### 개념

- 비동기 요청을 **명령형**으로 수행하고, `isLoading`·`error`·`data` 상태를 직접 관리합니다.
- 간단한 화면·독립적인 요청에는 직관적이며 학습 장벽이 낮습니다.

### 예시

```tsx
import { useEffect, useState } from "react";

function Users() {
  const [data, setData] = useState<string[] | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let alive = true;
    setIsLoading(true);
    fetch("/api/users")
      .then((r) => r.json())
      .then((json) => {
        if (alive) setData(json);
      })
      .catch((e) => {
        if (alive) setError(e);
      })
      .finally(() => {
        if (alive) setIsLoading(false);
      });
    return () => {
      alive = false;
    };
  }, []);

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorView error={error} />;
  if (!data) return null;
  return <UserList users={data} />;
}
```

### 장단점

- **장점**: 제어권이 높고, 요청 취소/재시도/병합 등 로직을 자유롭게 구성 가능.
- **단점**: 요청이 늘수록 `isLoading` 조합·조건부 렌더링이 복잡해지고, **N개의 스피너**가 난립하기 쉽습니다.

---

## 2) React Suspense 기반

### 개념

- 컴포넌트가 **Promise를 던지는 동안** 해당 서브트리 대신 `fallback`을 렌더링합니다.
- 상위의 **Suspense 경계**가 로딩 UI를 **선언적**으로 관리합니다.
- React 18부터 **Streaming SSR** 및 **`<Suspense>` 분할**과 잘 맞습니다.

### 예시(리소스 래퍼 패턴)

```tsx
// 간단 리소스
function createUserResource() {
  let status: "pending" | "success" | "error" = "pending";
  let result: any;
  const suspender = fetch("/api/users")
    .then((r) => r.json())
    .then((d) => {
      status = "success";
      result = d;
    })
    .catch((e) => {
      status = "error";
      result = e;
    });

  return {
    read() {
      if (status === "pending") throw suspender; // Suspense로 위임
      if (status === "error") throw result; // ErrorBoundary로 위임
      return result;
    },
  };
}

const userResource = createUserResource();

function Users() {
  const data = userResource.read();
  return <UserList users={data} />;
}

export default function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <Users />
    </Suspense>
  );
}
```

> 실제 프로젝트에서는 **TanStack Query(React Query)**, **Relay**, **Next.js Server Components** 등 **Suspense 호환 데이터 계층**을 사용하시는 것을 권장드립니다.

### 장단점

- **장점**
  - 로딩 UI를 **경계 단위**로 묶어 일관된 UX 제공(서브트리 단위 스켈레톤).
  - **중첩 Suspense**로 부분 준비 완료 영역부터 **점진적 표시**(스트리밍 SSR과 시너지).
  - 오류는 `ErrorBoundary`, 로딩은 `Suspense`로 **관심사 분리**.
- **단점**
  - **Promise 기반 소스** 필요(일반 `fetch`는 래핑 또는 라이브러리 필요).
  - 경계가 많으면 **fallback 깜빡임**·중첩 타이밍 이슈 → 경계 설계와 **지연(fallbackDelay)**·**placeholders** 조정 필요.

---

## 3) 기능 차이 한눈에 보기

| 항목           | useEffect + state      | Suspense                                  |
| -------------- | ---------------------- | ----------------------------------------- |
| 패러다임       | 명령형 로딩 제어       | 선언적 경계 기반 로딩                     |
| 로딩 UI 범위   | 컴포넌트별 조건부 렌더 | 경계 단위 서브트리                        |
| 다중 요청 조율 | 직접 상태 조합 필요    | 경계/중첩으로 자연스런 조율               |
| 오류 처리      | try/catch, 상태 분기   | ErrorBoundary로 위임                      |
| SSR/스트리밍   | 별도 구현              | React 18 스트리밍 SSR 친화                |
| 의존성         | React 기본만으로 가능  | Promise/thenable, 호환 데이터 계층 필요   |
| 미세 제어      | 매우 세밀              | 경계 중심(세밀 제어는 데이터 계층에 위임) |

---

## 4) 언제 무엇을 선택할까요

- **작고 단순한 화면, 단일 요청**: `useEffect + isLoading`이 가장 단순합니다.
- **화면 단위 스켈레톤, 부분적 점진 표시, SSR 연동**: **Suspense 우선**을 권장드립니다.
- **리스트+세부, 위젯 병렬 로딩**: 영역별 **Suspense 경계 분할**로 UX 정렬.
- **데이터 갱신/리페치 제어**가 많은 앱: **TanStack Query** 등과 **Suspense 모드**를 병행.

---

## 5) 실무 팁

1. **경계 설계**: 상단 레이아웃(헤더/내비)은 고정 렌더, 데이터 영역만 `Suspense`로 감싸 **스켈레톤 일관성**을 유지하십시오.
2. **깜빡임 완화**: 라이브러리의 `staleTime`, `placeholderData`, `suspense: true`, **fallback 지연**을 활용하십시오.
3. **에러/로딩 분리**: `ErrorBoundary`와 `Suspense`를 **형제**로 배치하시고, 오류 재시도 UI를 제공하십시오.
4. **SSR 연동**: Next.js App Router에서 **Server Components**로 가능한 데이터는 서버에서 해결하고, **클라이언트 컴포넌트**만 상호작용을 담당하게 하여 **hydration 비용**을 줄이십시오.
5. **혼합 전략**: 초기 로딩은 `Suspense`, 상호작용 후의 세밀한 리페치는 **명령형 제어(useEffect/Query)** 로 보완하는 하이브리드를 고려하십시오.

---

## 6) 간단 비교 예제(React Query)

```tsx
// Suspense 모드
function Users() {
  const { data } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
    suspense: true,
  });
  return <UserList users={data} />;
}

export default function Page() {
  return (
    <ErrorBoundary fallback={<ErrorView />}>
      <Suspense fallback={<Skeleton />}>
        <Users />
      </Suspense>
    </ErrorBoundary>
  );
}
```

```tsx
// 전통 모드
function Users() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });
  if (isLoading) return <Skeleton />;
  if (isError) return <ErrorView error={error} />;
  return <UserList users={data!} />;
}
```

---

## 7) 결론

- `useEffect + state`는 **로컬 제어가 쉬운** 반면, 규모가 커지면 **로딩 UI 조율 비용**이 커집니다.
- `Suspense`는 **경계 기반·선언적 로딩**으로 **대규모 UI**에서의 일관성과 SSR/스트리밍에 강점이 있습니다.
- 실무에서는 **데이터 계층(React Query/Relay/Server Components)** 를 활용해 Suspense를 도입하고, **세밀한 리페치 로직**은 기존 방식과 **혼합**하여 최적의 사용자 경험을 설계하시길 권장드립니다.
