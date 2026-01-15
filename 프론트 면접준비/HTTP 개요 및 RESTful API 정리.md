# HTTP 개요 및 RESTful API 정리

> 한눈 요약: **HTTP는 웹에서 요청–응답으로 데이터를 주고받는 애플리케이션 계층 프로토콜**입니다. 기본 특성은 **무상태성(stateless)** 이며, 전송 계층은 **HTTP/1.1·2에서는 TCP**, **HTTP/3에서는 QUIC(UDP 기반)** 을 사용합니다. 보안을 적용하면 **HTTPS(TLS 상의 HTTP)** 가 됩니다.

---

## 1) HTTP란 무엇인가

- **정의**: 클라이언트(브라우저 등)와 서버가 **요청(Request)–응답(Response)** 으로 통신하는 규약.
- **전달 포맷**: HTML, JSON, 이미지 등 어떤 바이트든 전송 가능.
- **구성 요소**
  - **메서드**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` 등
  - **상태 코드**: `200 OK`, `201 Created`, `204 No Content`, `301/302 Redirect`, `304 Not Modified`, `400/401/403/404`, `500/502/503` 등
  - **헤더**: `Content-Type`, `Accept`, `Authorization`, `Cache-Control`, `ETag`, `Cookie`, `Set-Cookie` 등
  - **바디**: 실제 페이로드(예: JSON, HTML, 파일 바이너리)

---

## 2) 핵심 특성

- **무상태성(Stateless)**: 각 요청은 **서버가 과거 상태를 기억하지 않는** 전제에서 독립 처리.
- **연결 관리**
  - **HTTP/1.1**: 기본 **keep-alive**(지속 연결)로 여러 요청을 하나의 TCP 연결에서 순차 처리.
  - **HTTP/2**: **단일 연결 다중화(multiplexing)**, 헤더 압축(HPACK)으로 지연 감소.
  - **HTTP/3**: **QUIC(UDP) 기반**. 전송 계층 단계에서 **헤드-오브-라인(HoL) 블로킹** 완화, 지연 내성 향상.
- **캐시 친화적**: `Cache-Control`, `ETag`, `Last-Modified` 등으로 네트워크 비용 절감.
- **콘텐츠 협상**: `Accept`, `Accept-Language`, `Accept-Encoding` 등으로 표현·언어·압축 방식 협상.

---

## 3) HTTPS

- **정의**: TLS 위에서 HTTP를 운반하는 보안 프로토콜.
- **효과**: 기밀성(암호화), 무결성(위·변조 방지), 서버 인증(인증서 기반).
- **운영 팁**: HSTS 적용, 최신 TLS 버전 사용, 안전한 암호군 선택, `Secure`, `HttpOnly`, `SameSite` 쿠키 설정.

---

## 4) 요청/응답 예시

### 요청

```
GET /api/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer <token>
```

### 응답

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=60

{"id":42,"name":"Alice"}
```

---

## 5) RESTful API란

- **REST 원칙을 따르는 API 설계 스타일**.
- **핵심 규칙**
  - **자원 식별**: URI로 자원 표현(`/users/42`).
  - **행위 표준화**: HTTP 메서드로 의도 표현(`GET` 조회, `POST` 생성, `PUT/PATCH` 수정, `DELETE` 삭제).
  - **무상태성**: 요청마다 필요한 컨텍스트를 포함, 세션 상태는 서버에 저장하지 않음.
  - **일관된 인터페이스**: 표현(대개 JSON), 상태 코드·헤더 활용으로 예측 가능성 확보.
  - **캐시 가능성**: 적절한 캐시 헤더로 성능 개선.
- **예시**
  - `GET /products` 목록 조회
  - `POST /products` 생성
  - `GET /products/{id}` 단건 조회
  - `PATCH /products/{id}` 부분 수정
  - `DELETE /products/{id}` 삭제

---

## 6) 실무 모범 사례

- **명확한 상태 코드/헤더 사용**: 오류는 `4xx/5xx`, 성공은 `2xx` 계열 일관 적용.
- **버전 관리**: `/v1` 같은 URI 버전 혹은 `Accept` 헤더 기반 버전 협상.
- **보안**: HTTPS 강제, OAuth2/OIDC 토큰, 최소 권한 원칙, 입력 검증.
- **성능**: 캐시 정책, 압축(`Content-Encoding: gzip/br`), HTTP/2·3 활성화, ETag 기반 조건부 요청.
- **가시성**: 표준 로그·추적 ID(`Traceparent`)로 관측성 확보.

---

## 7) 요약

- **HTTP**: 웹의 기본 언어. 요청–응답, 무상태성, 헤더·바디로 구성.
- **HTTPS**: HTTP에 TLS를 더해 안전한 통신 보장.
- **RESTful API**: URI로 자원 식별, HTTP 메서드로 행위 표현, 무상태·캐시·일관 인터페이스 지향.
