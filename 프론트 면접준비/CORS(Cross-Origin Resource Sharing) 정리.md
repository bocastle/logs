# CORS(Cross-Origin Resource Sharing) 정리

> 요약: 브라우저의 **동일 출처 정책(SOP)** 으로 인해 다른 출처의 응답 접근이 제한됩니다. **CORS** 는 서버가 특정(또는 모든) 출처에 대해 **명시적으로 허용**하는 표준 메커니즘입니다. 핵심은 **서버가 올바른 응답 헤더**를 보내도록 구성하는 것입니다.

---

## 1) 용어와 배경

- **출처(Origin)**: `프로토콜 + 호스트 + 포트` 조합. 셋 중 하나라도 다르면 **교차 출처**입니다.  
  예) `https://app.example.com:443` 와 `https://api.example.com:443` → **다른 출처**

- **SOP(Same-Origin Policy)**: 스크립트가 **다른 출처의 리소스 응답에 접근**하는 것을 제한하는 브라우저 보안 정책.

  - 주 목적: **교차 출처 응답의 무단 읽기**를 막아 **데이터 탈취** 위험을 낮춤.
  - 결과적으로 **CSRF의 위력도 낮아지지만**, CSRF 자체는 주로 **자동 전송 쿠키** 등과 결합될 때 문제가 됩니다.

- **CORS**: 서버가 특정 출처에 대해 접근을 **선별적 허용**하기 위한 규칙(응답 헤더 집합).

---

## 2) 요청 유형

### 2.1 Simple Request (사전 요청 없음)

다음 조건을 모두 만족하면 **Preflight** 없이 바로 본요청을 보냅니다.

- 메서드: `GET`, `HEAD`, `POST`
- 커스텀 헤더 없음(브라우저가 허용하는 안전한 헤더만)
- `Content-Type` 이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 중 하나

브라우저는 응답의 `Access-Control-Allow-Origin`(ACAO)을 확인해 접근 허용 여부를 판정합니다.

### 2.2 Preflight Request (사전 요청)

- 위 조건을 벗어나면 브라우저가 **`OPTIONS`** 로 사전 요청을 보냄.
- 사전 요청 헤더
  - `Origin`: 요청 출처
  - `Access-Control-Request-Method`: 실제 메서드
  - `Access-Control-Request-Headers`: 실제에 포함될 커스텀 헤더 목록
- 서버는 응답에 다음을 포함해야 합니다.
  - `Access-Control-Allow-Origin`: 허용 출처(정확한 Origin 또는 `*`)
  - `Access-Control-Allow-Methods`: 허용 메서드 목록
  - `Access-Control-Allow-Headers`: 허용 헤더 목록
  - `Access-Control-Max-Age`: 사전 요청 결과 캐시 시간(초)

### 2.3 Credentialed Request (인증 정보 동반)

- 쿠키/HTTP 인증/클라이언트 인증서를 동반하는 요청.
- 반드시 아래 모두 충족해야 함:
  - `Access-Control-Allow-Credentials: true`
  - `Access-Control-Allow-Origin` 에 **`*` 금지**(정확한 Origin 값으로 반환)
  - 클라이언트 측 `fetch`/XHR 에서 `credentials: 'include'` 설정

---

## 3) 핵심 응답 헤더 요약

| 헤더                               | 설명                                                  | 주의 사항                            |
| ---------------------------------- | ----------------------------------------------------- | ------------------------------------ |
| `Access-Control-Allow-Origin`      | 허용 출처 명시(`https://app.example.com` 또는 `*`)    | **Credentials와 `*` 동시 사용 불가** |
| `Access-Control-Allow-Methods`     | 허용 메서드(예: `GET, POST, PUT`)                     | Preflight 응답에 포함                |
| `Access-Control-Allow-Headers`     | 허용 헤더(예: `Authorization, Content-Type`)          | Preflight 응답에 포함                |
| `Access-Control-Allow-Credentials` | 인증 정보 허용 여부                                   | `ACAO:*` 와 함께 쓰면 차단           |
| `Access-Control-Expose-Headers`    | 스크립트에서 읽을 수 있는 응답 헤더 화이트리스트 확장 | 기본 노출 목록 외 헤더가 필요할 때   |
| `Access-Control-Max-Age`           | Preflight 결과 캐시 시간(초)                          | 너무 길면 정책 변경 반영 지연        |
| `Vary: Origin`                     | 캐시가 Origin 별로 응답을 구분하도록 지시             | 다중 출처 허용 시 필수적             |

---

## 4) 서버 설정 예시

### 4.1 Node/Express

```js
import express from "express";
import cors from "cors";

const app = express();
app.use(
  cors({
    origin: ["https://app.example.com", "https://admin.example.com"],
    credentials: true, // Access-Control-Allow-Credentials: true
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    maxAge: 600,
  })
);

app.get("/api/data", (req, res) => res.json({ ok: true }));
app.listen(3000);
```

### 4.2 Spring Boot (Java)

```java
@Configuration
public class CorsConfig {
  @Bean
  public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
      @Override
      public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
          .allowedOrigins("https://app.example.com", "https://admin.example.com")
          .allowedMethods("GET","POST","PUT","DELETE")
          .allowedHeaders("Content-Type","Authorization")
          .allowCredentials(true)
          .maxAge(600);
      }
    };
  }
}
```

---

## 5) 클라이언트 주의점

- **브라우저만 CORS 강제**: 서버↔서버 통신(백엔드→백엔드)은 CORS에 걸리지 않습니다.
- `fetch` 예시

```ts
await fetch("https://api.example.com/data", {
  method: "GET",
  credentials: "include", // 쿠키 포함 시
  headers: { Accept: "application/json" },
});
```

- 프록시/개발 서버를 활용해 **동일 출처로 라우팅**(개발 편의성)
- Preflight를 줄이려면: 불필요한 커스텀 헤더 제거, Simple Request 조건을 만족하도록 설계

---

## 6) 보안 관점 정리

- **CORS는 접근 제어(응답 읽기 권한) 정책**이지, **방화벽/인증 대체**가 아님.
- **민감 데이터 노출 금지**: 허용 출처 목록을 최소화하고, 와일드카드 사용을 신중히.
- **XSS 방어**(CSP/인코딩) 없이는, 페이지 내 JS가 탈취될 경우 CORS 허용도 무력화될 수 있음.
- CSRF 방어는 **SameSite 쿠키**, **CSRF 토큰**, **Origin/Referer 검증** 등과 **함께** 설계.

---

## 7) 트러블슈팅 체크리스트

- [ ] 응답에 `Access-Control-Allow-Origin` 이 정확히 설정되는가?(`*` 또는 요청 Origin 복사)
- [ ] `Vary: Origin` 을 넣어 CDN/프록시 캐시가 Origin 별로 분리되는가?
- [ ] Credentials 사용 시 `ACAO:*` 를 피하고, 클라이언트 `credentials: 'include'` 를 설정했는가?
- [ ] Preflight 응답에 **Methods/Headers/Max-Age** 가 정확한가?
- [ ] 서버/프록시 레벨에서 **CORS 헤더가 덮어써지거나 제거**되지 않는가?
- [ ] 개발·스테이징·프로덕션 **허용 출처 목록**이 환경별로 올바른가?

---

## 결론

- **SOP** 는 교차 출처 응답 접근을 제한해 데이터 탈취를 어렵게 만드는 핵심 브라우저 보안 모델입니다.
- **CORS** 는 SOP를 **안전하게 완화**하기 위한 서버 주도형 허용 정책으로, 올바른 **응답 헤더 구성**과 **캐시/크리덴셜 정책**이 관건입니다.
- 인증·권한·CSRF/XSS 방어 등 **다른 보안 대책**과 함께 종합적으로 설계해야 합니다.
