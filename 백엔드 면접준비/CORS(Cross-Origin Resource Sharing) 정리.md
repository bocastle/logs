# CORS(Cross-Origin Resource Sharing) 정리

> **정의**: 서로 다른 **출처(Origin: 스킴+호스트+포트)** 간의 리소스 접근을 **브라우저가 제어**하기 위한 표준 메커니즘입니다. 기본 정책인 **SOP(Same‑Origin Policy)** 를 확장하여, 서버가 **명시적으로 허용**한 범위에 한해 교차 출처 요청을 허용합니다.

---

## 1) 왜 필요한가 — SOP와 CORS의 관계

- **SOP**: 브라우저가 보안상 기본으로 **동일 출처만 접근 허용**. CSRF 등 공격 완화에 효과적.
- **한계**: 현대 웹은 API·CDN·서브도메인 등 **교차 출처** 호출이 빈번.
- **CORS**: 서버가 **허용 정책을 응답 헤더**로 전달하면, 브라우저가 이를 검증해 **선택적으로 허용**.

---

## 2) 동작 개요(세 가지 시나리오)

### 2.1 Simple Request

- 조건을 만족하는 **단순 요청**은 **사전 검사 없이** 바로 전송됩니다.
- 조건
  - 메서드: `GET`, `HEAD`, `POST`
  - 임의 헤더 미포함(허용: `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `Range`)
  - `Content-Type`: `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`
- 브라우저는 요청의 `Origin` 과 응답의 `Access-Control-Allow-Origin` 을 비교하여 허용 여부를 판단합니다.

### 2.2 Preflight Request

- **단순 요청 조건을 벗어나면** 브라우저가 **본 요청 전에** `OPTIONS` 로 **사전 요청**을 보냅니다.
- 요청 헤더
  - `Origin`: 요청자의 출처
  - `Access-Control-Request-Method`: 본 요청 메서드
  - `Access-Control-Request-Headers`: 본 요청에 포함될 커스텀 헤더 목록
- 응답 헤더
  - `Access-Control-Allow-Origin`: 허용 출처(정확한 값 또는 `*`)
  - `Access-Control-Allow-Methods`: 허용 메서드 목록
  - `Access-Control-Allow-Headers`: 허용 커스텀 헤더 목록
  - `Access-Control-Max-Age`: 프리플라이트 결과 캐시 시간(초)

### 2.3 Credentialed Request(인증 동반 요청)

- 쿠키/HTTP 인증/`Authorization` 헤더 등 **자격 증명**을 동반한 요청.
- 조건
  - 요청에 `credentials` 설정(fetch/XHR): `include`(쿠키 포함) 또는 `same-origin`
  - 응답에 `Access-Control-Allow-Credentials: true`
  - **주의**: 이 경우 `Access-Control-Allow-Origin` 에 **와일드카드 `*` 사용 불가**, 반드시 **정확한 출처**를 지정해야 합니다.

---

## 3) 주요 헤더 요약

| 방향            | 헤더                               | 설명                                                  |
| --------------- | ---------------------------------- | ----------------------------------------------------- |
| 요청            | `Origin`                           | 요청을 보낸 문서의 출처                               |
| 요청(Preflight) | `Access-Control-Request-Method`    | 본 요청의 메서드                                      |
| 요청(Preflight) | `Access-Control-Request-Headers`   | 본 요청의 커스텀 헤더 목록                            |
| 응답            | `Access-Control-Allow-Origin`      | 허용 출처(`*` 또는 특정 도메인)                       |
| 응답            | `Access-Control-Allow-Methods`     | 허용 메서드 목록                                      |
| 응답            | `Access-Control-Allow-Headers`     | 허용 커스텀 헤더 목록                                 |
| 응답            | `Access-Control-Expose-Headers`    | 자바스크립트에서 접근 가능하도록 **노출**할 응답 헤더 |
| 응답            | `Access-Control-Allow-Credentials` | 자격 증명 동반 요청 허용 여부                         |
| 응답            | `Access-Control-Max-Age`           | 프리플라이트 캐시 지속 시간(초)                       |

---

## 4) 서버 설정 예시

### 4.1 Express(Node.js)

```js
import express from "express";
import cors from "cors";

const app = express();

app.use(
  cors({
    origin: ["https://app.example.com", "https://admin.example.com"],
    credentials: true, // 쿠키/자격증명 허용
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization", "X-Request-Id"],
    exposedHeaders: ["X-Request-Id"], // 클라이언트에서 읽을 헤더
    maxAge: 600, // 프리플라이트 캐시(초)
  })
);

app.get("/api/data", (req, res) => res.json({ ok: true }));

app.listen(3000);
```

### 4.2 Spring Boot

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
          .allowedHeaders("*")
          .exposedHeaders("X-Request-Id")
          .allowCredentials(true)
          .maxAge(600);
      }
    };
  }
}
```

### 4.3 Nginx(리버스 프록시)

```nginx
location /api/ {
  if ($request_method = OPTIONS) {
    add_header Access-Control-Allow-Origin "https://app.example.com";
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
    add_header Access-Control-Allow-Headers "Content-Type, Authorization, X-Request-Id";
    add_header Access-Control-Allow-Credentials "true";
    add_header Access-Control-Max-Age 600;
    return 204;
  }

  add_header Access-Control-Allow-Origin "https://app.example.com" always;
  add_header Access-Control-Allow-Credentials "true" always;
  add_header Access-Control-Expose-Headers "X-Request-Id" always;

  proxy_pass http://backend_upstream;
}
```

---

## 5) 흔한 오해/주의 사항

- **CORS는 브라우저 보안 모델**입니다. 백엔드-백엔드(서버-서버) 통신에는 적용되지 않습니다.
- **클라이언트에서 헤더를 임의로 추가**해도, 최종 판단은 **브라우저가 서버 응답 헤더**를 보고 결정합니다.
- `Access-Control-Allow-Origin: *` 와 **Credentials 동시 사용 불가**.
- 프리플라이트를 과도하게 유발하는 **커스텀 헤더/메서드**는 가능하면 표준 헤더·단순 요청 조건으로 조정하십시오.
- 서브도메인도 **출처가 다르면 다른 Origin**입니다. 필요 시 와일드카드 매칭이 아닌 **명시 목록/동적 반영**을 사용하십시오.

---

## 6) 트러블슈팅 체크리스트

- [ ] **정확한 Origin** 이 `Access-Control-Allow-Origin` 에 반영되는가
- [ ] Credentials 사용 시 `Allow-Origin` 이 `*` 가 아닌가
- [ ] 필요한 메서드/헤더가 `Allow-Methods/Allow-Headers` 에 포함됐는가
- [ ] 프리플라이트 응답이 **204/200** 으로 정상 반환되는가, `Max-Age` 로 캐시되는가
- [ ] 프록시/로드밸런서가 CORS 헤더를 **제거/변조**하지 않는가
- [ ] 브라우저 개발자도구 **Network 탭**에서 **Request/Response 헤더**를 직접 확인했는가
- [ ] 캐시/CDN이 오래된 CORS 헤더를 **서빙**하고 있지 않은가

---

## 7) 요약

- CORS는 SOP의 제한을 **서버가 정의한 허용 정책**으로 완화하는 메커니즘입니다.
- 브라우저는 `Origin` ↔ `Access-Control-Allow-Origin` 을 비교하고, 필요 시 **프리플라이트**로 안전성을 검증합니다.
- **인증 요청**에는 `Allow-Credentials: true` 와 **정확한 Origin 지정**이 필수이며, 운영 환경에서는 **최소 권한 원칙**과 **프리플라이트 캐시**로 성능과 보안을 함께 확보하십시오.
