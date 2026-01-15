# SSR(Server‑Side Rendering) 정리

> 정의: **SSR**은 서버에서 **완성된 HTML**을 만들어 응답하고, 클라이언트는 이를 즉시 렌더링한 뒤 **hydration**으로 상호작용(이벤트 리스너 등)을 활성화하는 방식입니다. **CSR**은 빈 골격 HTML과 JS 번들을 내려받아 **브라우저에서 UI를 구성**합니다.

---

## 1) 렌더링 방식 비교

| 구분           | SSR                                 | CSR                        |
| -------------- | ----------------------------------- | -------------------------- |
| 초기 응답      | 서버가 HTML 생성(데이터 포함)       | 최소 HTML + JS 번들 링크   |
| 초기 체감 속도 | **빠름**(즉시 픽셀 표시)            | 번들 다운로드/실행 후 표시 |
| 상호작용 준비  | hydration 이후 가능                 | 번들 실행 직후 가능        |
| SEO            | **유리**(완성 HTML 크롤러에 노출)   | 사전 렌더 없으면 불리      |
| 서버 부하/비용 | 요청마다 렌더 시 **높음**           | 낮음                       |
| 라우팅 감각    | 서버 라우팅 중심(전통)              | 클라이언트 라우팅 중심     |
| 복잡도         | SSR+CSR 혼합 시 **상대적으로 높음** | 단순(순수 CSR)             |

보완적 대안: **SSG**(빌드 시 정적 생성), **ISR**(정적+주기적 재생성), **스트리밍 SSR**(부분 HTML 스트리밍), **엣지 SSR**(Edge 런타임).

---

## 2) SSR 플로우(개념)

```
Client → (요청) → Server 렌더(데이터 fetch) → HTML 응답
      ← HTML 표시 (즉시 픽셀) ←
      → JS 번들 다운로드/실행 → hydration(이벤트 부착) → 상호작용 가능
```

- **TTV**(Time To View): HTML 표시까지 빠름
- **TTI**(Time To Interactive): hydration 완료 전까지는 **클릭 무응답 구간**이 생길 수 있음

---

## 3) 장점

1. **SEO에 유리**: 완성 HTML 기반 색인·미리보기(OG 태그 등) 용이
2. **빠른 초기 렌더**: 저사양·저속 네트워크 환경에서 체감 향상
3. **보안/비밀키 보호**: 서버에서의 데이터 호출·토큰 보관
4. **일관된 데이터**: 서버 시점에서의 집계·접근 제어 적용 용이

---

## 4) 단점/주의사항

1. **hydration 지연**: TTV ↔ TTI 간의 간극으로 초기 상호작용 지연
2. **서버 비용/복잡도 증가**: 요청마다 렌더, 캐싱·CDN·데이터 경로 설계 필요
3. **복합 아키텍처**: SSR과 CSR 혼용 시 코드 경계(서버/클라이언트) 관리 필요
4. **데이터 패칭 폭포**(waterfall) 위험: 서버 렌더 단계에서 연쇄 fetch 발생 가능
5. **Hydration 불일치**: 서버/클라이언트 렌더 결과 차이로 경고/런타임 오류

---

## 5) Next.js 관점(요약)

### App Router(권장 최신 아키텍처)

- 기본이 **Server Components** → 불필요한 JS를 **클라이언트로 보내지 않음**(hydration 비용 감소)
- 데이터 패칭: 컴포넌트에서 `async`/`fetch()` 사용
  - `fetch(url, { cache: 'no-store' })` → **매 요청 SSR**
  - `fetch(url, { next: { revalidate: 60 } })` → **ISR(60초)**
- **Streaming/선점 렌더**: `Suspense`로 **부분 스트리밍**(TTFB/LCP 개선)
- 상호작용 필요한 컴포넌트에 한해서 `"use client"` 로 클라이언트 컴포넌트 분리

```tsx
// app/page.tsx (Server Component)
import ProductList from "./product-list";

export default async function Page() {
  const res = await fetch("https://api.example.com/products", {
    next: { revalidate: 60 },
  });
  const products = await res.json();
  return <ProductList products={products} />;
}

// app/product-list.tsx (Client Component: 상호작용 필요)
("use client");
export default function ProductList({ products }: { products: any[] }) {
  // 정렬/필터 UI 등 클라이언트 상호작용
  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

### Pages Router(레거시/병행)

- `getServerSideProps` 로 **요청 시** 데이터 패칭 및 SSR
- `getStaticProps` + `getStaticPaths` 로 SSG/ISR

```ts
// pages/products.tsx
export async function getServerSideProps() {
  const res = await fetch("https://api.example.com/products");
  const data = await res.json();
  return { props: { data } };
}
export default function Products({ data }: { data: any[] }) {
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

---

## 6) SSR 성능/안정성 최적화

- **스트리밍 SSR + Suspense**: 비동기 경계 분할로 **TTFB/LCP 개선**
- **캐싱 계층화**: CDN(Cache‑Control), ISR(revalidate), 서버 캐시, DB 캐시
- **데이터 병렬 패칭**: Promise.all/React Server Components의 병렬 fetch
- **코드 분할**: 상호작용 필요한 영역만 **클라이언트 JS** 전송
- **리소스 힌트**: `preload`/`dns-prefetch`/`preconnect` 로 네트워크 준비
- **Hydration 비용 절약**: RSC 활용, `use client` 최소화, 이벤트 핸들러 범위 줄이기
- **불일치 방지**: 서버/클라이언트 동일한 지역·시간·랜덤 소스 사용, `suppressHydrationWarning`는 최후 수단

---

## 7) 언제 SSR을 선택할까

- **검색 유입/공유 미리보기**가 중요한 블로그, 커머스, 미디어 페이지
- **초기 렌더 체감이 중요한** 랜딩/마케팅 페이지
- **접근 제어/퍼스널라이제이션**을 서버에서 수행해야 하는 경우
- 반면, **순수 앱형 대시보드**·실시간 상호작용 위주라면 CSR/하이브리드가 효율적일 수 있습니다

---

## 8) 체크리스트

- [ ] 어떤 경로가 **SSR/SSG/ISR/CSR**인지 **명확히 구분**했는가
- [ ] **데이터 캐싱 전략**(CDN/ISR/서버 캐시)을 정의했는가
- [ ] **스트리밍 SSR/Suspense**로 폭포형 대기를 줄였는가
- [ ] 클라이언트 JS를 **최소화**(RSC, 코드 분할)했는가
- [ ] **Hydration 지연 UX**(스켈레톤/플레이스홀더/낮은 우선순위 이벤트) 설계가 있는가
- [ ] **보안/비밀키**는 서버에서만 처리되는가
- [ ] **오류·장애 대응**: 서버 타임아웃·캐시 폴백·로깅/트레이싱 준비가 되어 있는가

---

## 9) 요약

- SSR은 **빠른 초기 표시와 SEO**에 유리하나, **hydration 지연/서버 비용/복잡도**를 수반합니다.
- Next.js 최신 구조는 **Server Components + Streaming SSR + ISR**로 단점을 완화합니다.
- 페이지 특성에 따라 **SSR/SSG/ISR/CSR**을 **하이브리드**로 조합하시기를 권장드립니다.
