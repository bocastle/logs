# ⚙️ Middleware란?

## 💡 개요

**Middleware(미들웨어)**는  
요청(Request)과 응답(Response) 사이에서 **중간 처리 로직을 수행하는 소프트웨어 레이어**입니다.  
즉, 클라이언트가 서버에 요청을 보내고 서버가 응답을 반환하기 전에,  
**공통적인 기능(로깅, 인증, 보안, 데이터 가공 등)**을 수행하는 역할을 합니다.

---

## 🧩 일반적인 동작 흐름

Client → [Middleware] → Route Handler → Response

Middleware는 요청이 실제 비즈니스 로직(컨트롤러)에 도달하기 전,  
공통적인 작업을 처리한 뒤 요청을 다음 단계로 넘깁니다.

---

## ⚙️ 주요 역할

| 역할                   | 설명                             |
| ---------------------- | -------------------------------- |
| **인증/인가 처리**     | 로그인 상태 확인, 권한 검증      |
| **로깅(Log)**          | 요청/응답 로그 저장              |
| **에러 핸들링**        | 예외 처리 및 에러 응답 변환      |
| **요청 데이터 전처리** | Body, Query, Header 검사 및 변환 |
| **응답 데이터 후처리** | 응답 구조 통일, 헤더 추가        |

---

## 🌐 프레임워크별 예시

### 🧱 Express.js

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // 다음 미들웨어 또는 라우터로 전달
});
```

## ⚛️ Next.js

Next.js의 Middleware는 middleware.ts 파일로 정의되며,
요청이 페이지나 API 라우트에 도달하기 전에 실행됩니다.

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  if (!req.cookies.get("auth")) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
  return NextResponse.next();
}
```

## 🚀 정리

| 구분             | 설명                                                 |
| ---------------- | ---------------------------------------------------- |
| **위치**         | 요청(Request)과 응답(Response) 사이                  |
| **목적**         | 공통 로직(인증, 로깅, 리다이렉션 등)을 중앙에서 처리 |
| **Next.js 특징** | Edge Runtime에서 실행되어 빠른 응답 속도 제공        |

---

💬 **한마디로 요약하자면:**  
**Middleware는 요청과 응답 사이의 “중간 관리자”**로,  
앱 전반에 걸친 공통 기능을 효율적으로 처리하기 위한 핵심 구성 요소입니다.
