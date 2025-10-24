# 🧱 Next.js 아키텍처 설명

Next.js는 React를 기반으로 하여 웹 애플리케이션을 더 쉽게 만들 수 있게 해주는 프레임워크이며, 내부적으로 여러 아키텍처적 특징을 갖고 있습니다. 아래에 주요 구성 및 흐름을 정리해드릴게요.

---

## ⚙️ 주요 구성 요소 및 특징

### 1. 파일 시스템 기반 라우팅 (File-System Based Routing)

- `pages/` 또는 `app/` 디렉토리 구조를 통해 URL 경로가 자동으로 매핑됩니다. :contentReference[oaicite:1]{index=1}
- 동적 라우트(dynamic route)나 중첩 라우트(nested route)도 디렉토리/파일명으로 정의 가능합니다. :contentReference[oaicite:2]{index=2}
- API 라우트(`pages/api/…`)를 통해 프론트엔드와 동일한 코드베이스에서 백엔드 엔드포인트도 정의할 수 있습니다. :contentReference[oaicite:3]{index=3}

### 2. 렌더링 전략 (Rendering Strategies)

Next.js는 다양한 렌더링 방식을 지원해서 상황에 맞게 선택할 수 있습니다. :contentReference[oaicite:4]{index=4}

- SSR (Server-Side Rendering): 요청 시점에 서버에서 HTML을 생성
- SSG (Static Site Generation): 빌드 시점에 HTML을 미리 생성
- ISR (Incremental Static Regeneration): 배포 후에도 정해진 조건에 따라 정적 페이지를 갱신
- CSR (Client-Side Rendering): 기존 React 방식처럼 클라이언트에서 렌더링  
  이 전략들을 혼합(hybrid)해서 사용할 수 있는 구조입니다. :contentReference[oaicite:5]{index=5}

### 3. 서버 컴포넌트 & 클라이언트 컴포넌트

- Next.js 13+ 버전부터는 **Server Components**와 **Client Components** 개념이 도입되어,  
  서버에서만 렌더링되는 컴포넌트와 클라이언트에서 실행되어야 하는 컴포넌트를 분리할 수 있습니다. :contentReference[oaicite:6]{index=6}
- 이를 통해 불필요한 클라이언트 자바스크립트 부담을 줄이고 퍼포먼스를 향상시키는 데 기여합니다.

### 4. 내부 최적화 및 빌드/런타임 구조

- Next.js는 자동 코드 스플리팅, 이미지 최적화(`next/image`), 폰트 최적화(`next/font`) 등 다양한 최적화 기능을 기본 제공합니다. :contentReference[oaicite:7]{index=7}
- 빌드 도구, 컴파일러 등을 직접 설정하지 않아도 되는 “제로 설정(zero-config)” 경험을 제공하며, 내부적으로 Rust 기반 컴파일러 등이 사용되기도 합니다. :contentReference[oaicite:8]{index=8}

---

## 🔄 요청 흐름(Request Flow) 개요

1. 사용자가 브라우저에서 URL로 요청을 보냅니다.
2. Next.js 서버 또는 Edge 런타임에서 해당 요청을 수신합니다.
3. 파일 시스템 기반 라우터가 요청된 경로에 해당하는 페이지 또는 API 핸들러를 찾습니다.
4. 필요한 경우 `getServerSideProps`, `getStaticProps` 또는 Server Component 등에서 데이터를 가져옵니다. :contentReference[oaicite:9]{index=9}
5. 페이지를 렌더링하고 HTML + 자바스크립트 번들을 생성해서 클라이언트에 전달합니다.
6. 클라이언트에서는 초기 HTML이 렌더링된 후, 필요에 따라 React 하이드레이션(hydration)을 수행하거나 클라이언트 컴포넌트가 실행됩니다.
7. 이후 클라이언트 내에서 리액트 라우팅, 인터랙션 등이 발생합니다.

---

## 🧭 정리

| 구분              | 설명                                                                 |
| ----------------- | -------------------------------------------------------------------- |
| **라우팅 방식**   | 파일 시스템 기반 자동 라우팅으로 직관적인 구조 제공                  |
| **렌더링 방식**   | SSR, SSG, ISR, CSR을 혼합해서 사용 가능                              |
| **컴포넌트 분리** | 서버 측과 클라이언트 측 컴포넌트를 분리함으로써 퍼포먼스 향상        |
| **최적화 기능**   | 이미지/폰트 최적화, 자동 코드 스플리팅 등 제공                       |
| **전체 아키텍처** | 프론트엔드 + API + 서버사이드 렌더링을 하나의 프레임워크로 통합 가능 |

---

Next.js는 React 기반 웹 애플리케이션을 **더 빠르게**, **더 효율적으로**, **더 확장 가능하게** 만들기 위한 아키텍처적 설계를 갖추고 있습니다.
