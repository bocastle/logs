# Streaming SSR 정리

## 개요

**Streaming SSR**(Streaming Server-Side Rendering)은 서버가 HTML을 모두 완성한 뒤 한 번에 응답하는 전통적 SSR과 달리, **준비된 조각부터 점진적으로 전송(Chunked/Streamed)** 하여 **TTFB(Time To First Byte)와 초기 가시성**을 개선하는 렌더링 방식입니다. React 18은 서버에서 HTML을 스트리밍하는 API를 제공하며, 대표적으로 Node.js 환경의 `renderToPipeableStream`(streams1)과 Web Streams 환경의 `renderToReadableStream`(streams2)을 지원합니다.

---

## 전통 SSR vs Streaming SSR vs CSR 비교

| 항목                    | 전통 SSR               | Streaming SSR              | CSR                        |
| ----------------------- | ---------------------- | -------------------------- | -------------------------- |
| 초기 표시 속도          | 보통(모든 데이터 대기) | 빠름(조각 단위 전송)       | 느림(번들 로드 및 실행 후) |
| 네트워크                | 단일 큰 응답           | **Chunked 전송**(점진적)   | 초기 JS 의존               |
| 상호작용 가능 시점(TTI) | SSR+Hydration 이후     | SSR 조각 + Hydration 분할  | 번들 실행 이후             |
| SEO 친화성              | 좋음                   | **매우 좋음**              | 라우터/미리 렌더 필요      |
| 구현 난이도             | 중                     | **높음(경계/동기화 설계)** | 중                         |
| 오류 격리/부분 로딩     | 한계                   | **Suspense 경계 기반**     | 라우팅/코드 스플리팅 의존  |

> 핵심: Streaming SSR은 **중요 콘텐츠 우선 노출**과 **부분 Hydration**으로 체감 성능을 올리지만, **데이터 동기화 및 일관성**을 매우 신중히 설계해야 합니다.

---

## React 18 서버 API 예시

### 1) Node.js(Express 등) – `renderToPipeableStream`

```tsx
import express from "express";
import { renderToPipeableStream } from "react-dom/server";
import App from "./App";

const app = express();

app.get("/", (req, res) => {
  const { pipe, abort } = renderToPipeableStream(<App />, {
    onShellReady() {
      res.setHeader("Content-Type", "text/html; charset=utf-8");
      // 프락시가 버퍼링하지 않도록 Transfer-Encoding 유지
      // Nginx 등에서는 proxy_buffering off 고려
      pipe(res);
    },
    onAllReady() {
      // 모든 경계가 준비된 뒤 호출(필요 시 사용)
    },
    onShellError(err) {
      res.statusCode = 500;
      res.setHeader("Content-Type", "text/plain; charset=utf-8");
      res.end("Internal Server Error");
    },
    onError(err) {
      console.error(err);
    },
  });

  // 백엔드 호출 지연 등으로 과도한 대기 방지
  setTimeout(() => abort(), 10000);
});

app.listen(3000);
```

### 2) Web Streams 환경(Cloudflare Workers 등) – `renderToReadableStream`

```tsx
import { renderToReadableStream } from "react-dom/server";
import App from "./App";

export default {
  async fetch(request: Request) {
    const stream = await renderToReadableStream(<App />);
    // shell 준비될 때까지 대기하면 첫 바이트 지연됨
    // 필요 시 controller.ready 사용
    return new Response(stream, {
      headers: { "Content-Type": "text/html; charset=utf-8" },
    });
  },
};
```

---

## 핵심 설계 포인트

### 1) **Suspense 경계 배치**

- **큰 페이지를 여러 Suspense 경계로 나누어** 상위(핵심) 섹션부터 먼저 flush.
- `fallback`은 가벼운 스켈레톤/로더를 사용하여 **layout shift 최소화**.
- 데이터 의존성이 큰 컴포넌트를 **하위 경계**로 내려 **요청 병렬화 → 빠른 Shell 표시**.

### 2) **Hydration 일관성 확보(불일치 방지)**

- 서버와 클라이언트가 **동일한 데이터 스냅샷**을 사용하도록 설계.
  - 예: TanStack Query `dehydrate()/Hydrate`로 **서버 캐시 → 클라이언트 복원**.
  - 환경 의존 값(시간, 랜덤, 로케일)은 **서버 결과를 직렬화**하여 클라이언트에 주입.
- **클라이언트 최초 렌더가 서버 HTML과 다른 출력**을 만들면 경고/재조정 발생.
  - 랜덤/날짜/`Math.random()`/`Date.now()`/`window` 의존 로직은 **useEffect 이후** 반영.

### 3) **전송/프락시 설정**

- **Chunked 전송**이 중간 프락시(Nginx, CDN)에서 **버퍼링**되지 않도록 설정.
  - Nginx: `proxy_buffering off;` / `gzip off;`(환경에 따라) / HTTP/1.1 유지 등.
- **압축**(gzip/br)은 스트림 호환 모드로 구성하여 **chunk flushing과 양립**.
- CDN 캐시를 쓸 경우, **부분 응답 캐싱 전략**(경계 구간 캐싱 불가)을 별도로 설계.

### 4) **에러/타임아웃 전략**

- `onShellError`에서 **초기 Shell 실패 시 대체 응답**.
- **경계별 오류 UI**: Suspense + Error Boundary로 **영역 단위 복구**.
- 백엔드 지연 시 `abort()`로 **부분 렌더 종료** 및 **대체 UI 스위치**.

### 5) **SEO/분석 호환**

- 검색엔진은 **서버 HTML을 우선 수집** → Streaming SSR 유리.
- 클라이언트 렌더에 의존하던 **메타태그/오픈그래프**는 **서버에서 출력**.
- 웹 분석 스크립트는 **Hydration 이후 지연 로드**로 TTI 영향 최소화.

---

## Hydration 불일치(서버-클라이언트 상태 차이) 원인과 해결

| 원인                        | 증상                  | 해결책                                                |
| --------------------------- | --------------------- | ----------------------------------------------------- |
| 랜덤/시간 의존 렌더         | 경고, DOM 재조정      | 서버 계산값을 JSON으로 주입 → 클라이언트 동일 값 사용 |
| 사용자별 상태/권한 차이     | 일부 섹션 re-render   | 쿠키/세션 토큰 기반 SSR, 초기 상태 직렬화             |
| 데이터 레이스(요청 시점 차) | UI 깜빡임             | Suspense 경계로 분리, 중요 데이터는 서버에서 확정     |
| i18n/로케일 포맷 차         | 숫자/날짜 포맷 불일치 | 서버와 동일 로케일/포맷터 사용, 직렬화된 포맷 전달    |
| 레이아웃 시프트             | LCP 저하              | 고정 크기 스켈레톤, 이미지 `width/height` 명시        |

---

## TanStack Query와의 연동(권장 패턴)

1. 서버에서 쿼리를 미리 **prefetch** → `dehydrate(queryClient)`로 **직렬화**.
2. HTML 내 **스크립트 태그로 초기 캐시 삽입**.
3. 클라이언트에서 `Hydrate`로 **동일 캐시 복원** → **불일치 최소화/즉시 데이터 사용**.
4. 필요 시 `staleTime` 조절로 **즉시 재검증 여부**를 제어.

```tsx
// 서버
await queryClient.prefetchQuery(["post", id], () => fetchPost(id));
const dehydrated = dehydrate(queryClient);
// dehydrated 상태를 <script>로 주입

// 클라이언트
<Hydrate state={window.__DEHYDRATED_STATE__}>
  <PostDetail id={id} />
</Hydrate>;
```

---

## Next.js(App Router)에서의 포인트

- **Server Components + Streaming** 조합으로 **데이터 fetch → 서버에서 처리** 후 스트리밍.
- `loading.tsx`(경계 역할), `Suspense` 중첩, `route segment`별 **병렬 데이터 로딩**.
- Edge 런타임/Node 런타임에서 **Web Streams/Node Streams** 각각 최적화.
- 캐시 정책(`revalidate`, `fetch`의 `cache`/`next` 옵션)으로 **데이터 신선도/성능 균형**.

---

## 언제 Streaming SSR을 쓰면 좋은가

- LCP/LCP 요소가 **서버 데이터 의존**이며 **초기 가시성**이 중요한 페이지.
- **대형 페이지/다수의 데이터 소스**를 조각으로 나눠 먼저 보여주고 싶은 경우.
- Cloud/Edge 런타임에서 **지리적 근접성과 스트리밍**을 함께 활용하고 싶은 경우.

## 피해야 할/신중할 상황

- **중간 프락시가 강제 버퍼링**하여 스트림 이점이 사라지는 인프라.
- 강한 **완전 캐싱(정적 HTML 캐시)**을 우선시하는 페이지(정적 생성이 더 적합).
- 데이터 일관성 요구가 매우 엄격하고 **부분 UI 허용이 어려운** 업무 화면.

---

## 체크리스트

- [ ] Suspense 경계를 의미 있게 분리했는가(핵심 섹션 우선 flush)?
- [ ] 서버/클라이언트 **동일 데이터 스냅샷**을 보장하는가?
- [ ] 프락시/CDN이 **chunked 전송을 버퍼링하지 않도록** 설정했는가?
- [ ] 에러/타임아웃/대체 UI 정책이 있는가?
- [ ] 메타/OG 태그를 **서버에서 보장**하는가?
- [ ] 이미지 크기/레이지 로딩 등 **CLS/LCP** 최적화가 반영되었는가?

---

## 용어 메모

- **onShellReady**: 초기 Shell이 렌더 가능해질 때 호출(빠른 TTFB).
- **onAllReady**: 모든 경계가 준비된 뒤 호출(완전한 HTML 이후).
- **Abort**: 스트림을 중단하고 대체 UI로 전환.
- **Backpressure**: 소비자 처리 속도에 맞춘 스트림 조절.

---

## 결론

Streaming SSR은 **초기 체감 속도**와 **핵심 콘텐츠 우선 노출**을 크게 개선하지만, **데이터 일관성**과 **인프라 스트리밍 호환성**을 전제로 해야 합니다. Suspense 경계 설계, TanStack Query 직렬화/복원, 프락시 설정, 에러/타임아웃 전략을 함께 적용하면 **빠르고 안정적인 사용자 경험**을 구현할 수 있습니다.
