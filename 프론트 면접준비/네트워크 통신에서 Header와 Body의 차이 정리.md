
# 네트워크 통신에서 Header와 Body의 차이 정리

> 요약: **Header는 “어떻게 처리할지”에 대한 메타데이터**, **Body는 “무엇을 전송하는지”에 대한 실제 데이터**입니다. HTTP/2·3에서는 Header가 **HPACK/QPACK으로 압축**되어 프레임으로 전달되고, Body는 **DATA 프레임**으로 전송됩니다.

---

## 1) 개념 비교

| 구분 | Header | Body(Payload) |
|---|---|---|
| 의미 | 메시지 처리 지침(메타데이터) | 전송하려는 실제 데이터 |
| 예시 | `Content-Type`, `Authorization`, `Accept`, `Cache-Control`, `Cookie`, `Set-Cookie` | JSON, HTML, 이미지 바이너리, 폼 데이터 등 |
| 크기 특성 | 일반적으로 작고 짧음(수 KB 수준 권장) | 상대적으로 큼(수 KB~수 MB/GB) |
| 처리 시점 | 라우팅, 인증, 캐싱 정책 결정에 먼저 사용 | 애플리케이션 로직이 실제로 소비 |
| 캐싱·보안 | 캐시 정책·인증·CORS 등 정책 전달 | 민감 정보 자체는 가급적 포함 지양(암호화/토큰 등 고려) |

---

## 2) HTTP 메시지 구조 예시

### 요청(Request)
```
POST /api/orders HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Content-Length: 27

{"itemId":123,"qty":2}
```

### 응답(Response)
```
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-store

{"orderId":"A-20260104-001"}
```

- **Header**: 콘텐츠 유형, 인증, 캐시, 협상 정보 등을 전달
- **Body**: 실제 페이로드(JSON 등)

---

## 3) Header 세부
- **요청 공통**: `Host`, `Accept`, `Accept-Encoding`, `Accept-Language`, `User-Agent`
- **요청 인증/보안**: `Authorization`, `Cookie`, `Origin`, `Referer`
- **응답 공통**: `Content-Type`, `Content-Length`, `Cache-Control`, `ETag`, `Last-Modified`
- **쿠키 설정**: `Set-Cookie`(HttpOnly/Secure/SameSite 등 속성으로 보안 관리)
- **전송 방식**: HTTP/1.1의 `Transfer-Encoding: chunked`, HTTP/2·3은 프레이밍으로 대체
- **압축**: HTTP/2 **HPACK**, HTTP/3 **QPACK**

---

## 4) Body 세부
- **형식 지정**: `Content-Type`(예: `application/json`, `multipart/form-data`, `text/html`)
- **길이 지정**: `Content-Length` 또는 프레임/청크 기반 전송
- **비울 수 있음**: 예를 들어 `GET` 요청은 Body가 없어도 무방(규격상 허용되지만 일반적으로 사용하지 않음)

---

## 5) 크기 제한과 오류 코드
- **표준(RFC)에는 Header 크기 상한이 명시되어 있지 않음**. 대신 **서버/리버스 프록시/클라이언트**가 자체 제한을 둠.
  - 실무 기본값 예시: **8~16KB** 수준(Nginx/Apache/Node 런타임·프록시 등 환경별 상이)
- **초과 시 대표 반응**
  - **431 Request Header Fields Too Large**: 헤더(또는 특정 헤더 필드)가 너무 큰 경우에 흔히 사용
  - **400 Bad Request**: 일부 서버는 포괄적으로 처리
  - **413 Content Too Large**: 주로 **Body(페이로드)가 너무 큰 경우**에 사용됨
- 결론: 헤더가 과도하게 커지지 않도록 **쿠키/커스텀 헤더 최소화**, 필요 시 **세션/메타데이터는 서버 저장소**로 위임

---

## 6) 프로토콜·상태별 유의사항
- **HEAD 응답**, **204 No Content**, **304 Not Modified**: **Body가 없음**
- **CORS·캐시·콘텐츠 협상**: 전부 **Header 중심**으로 동작
- **HTTP/2·3**: 다중화 환경에서 헤더는 압축·프레이밍되어 전송 효율↑

---

## 7) 보안/성능 모범 사례
- **민감 정보는 Body/Header 모두에 평문 저장 지양**. 인증은 **Bearer 토큰/쿠키(HttpOnly, Secure, SameSite)** 등 정책 준수
- **헤더 최적화**: 불필요한 커스텀 헤더/쿠키 삭제, 쿠키 스코프·만료 관리
- **Body 최적화**: JSON 축소(minify), 바이너리 전송 시 압축/스트리밍, 대용량은 범위 전송 또는 분할 업로드 고려
- **캐시 활용**: `ETag`, `If-None-Match`, `Cache-Control`, `Vary`로 대역폭 절감

---

## 8) 한눈에 보는 차이
- **Header = 설명서**: “이 데이터는 무엇이고, 어떻게 처리해야 하는가”를 알려줍니다.
- **Body = 내용물**: 실제로 전송·소비되는 데이터 자체입니다.

