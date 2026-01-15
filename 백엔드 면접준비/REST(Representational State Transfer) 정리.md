# REST(Representational State Transfer) 정리

> 요약: **REST**는 네트워크 상에서 **자원(Resource)** 을 **표현(Representation)** 으로 식별하고, 표준 **HTTP 메서드**로 상태를 전이(Transfer)하는 **아키텍처 스타일**입니다. 보통 **URI로 자원을 명시**하고 **JSON**으로 표현을 주고받습니다.

---

## 1) 핵심 개념

- **자원(Resource)**: 서버가 관리하는 모든 대상(예: `orders`, `users`, `products`).
- **표현(Representation)**: 자원의 현재 상태를 전달하는 형태(JSON, XML 등).
- **상태 전이(State Transfer)**: `GET/POST/PUT/PATCH/DELETE` 등 메서드로 자원 상태를 바꾸거나 조회.

```http
GET /orders/42           # 주문 42 조회
POST /orders             # 주문 생성
PUT /orders/42           # 주문 42 전체 교체
PATCH /orders/42         # 주문 42 부분 수정
DELETE /orders/42        # 주문 42 삭제
```

---

## 2) REST 아키텍처 제약 조건

1. **클라이언트–서버 분리**: UI(클라이언트)와 데이터 저장/비즈니스 로직(서버) 분리.
2. **무상태(Stateless)**: 각 요청은 **자급자족**(필요한 인증/컨텍스트 포함)해야 함.
3. **캐시 가능(Cacheable)**: 응답은 `Cache-Control`, `ETag/If-None-Match` 등으로 캐시 제어.
4. **균일한 인터페이스(Uniform Interface)**: 일관된 URI 설계·표준 메서드·표준 상태코드.
5. **계층화 시스템(Layered System)**: 프록시/CDN/게이트웨이 등 중간 계층 허용.
6. _(선택)_ **Code on Demand**: 필요 시 서버가 실행 코드(스크립트)를 전송.

> 이 제약을 지킬수록 **확장성·독립성·가시성**이 높아집니다(완벽 준수는 선택).

---

## 3) URI/리소스 모델링 가이드

- **명사형 복수 자원명** 사용: `/orders`, `/users/{id}`
- **관계 자원**: `/users/{id}/orders`, `/orders/{id}/items`
- **필터/정렬/페이지네이션**: `GET /orders?status=PAID&sort=-createdAt&page=2&size=20`
- **컬렉션 조작**: `POST /orders`(생성), `GET /orders`(목록), `DELETE /orders?before=...`(배치 삭제)
- **부분 갱신**: `PATCH /orders/42`(JSON Merge Patch/JSON Patch 고려)

예시 페이로드

```json
// POST /orders
{
  "customerId": 1001,
  "items": [{ "productId": 77, "qty": 2 }]
}
```

---

## 4) HTTP 메서드·상태 코드 매핑

| 메서드 | 의미             |    멱등성 | 예시 상태코드           |
| ------ | ---------------- | --------: | ----------------------- |
| GET    | 조회             |         ✔ | 200, 304(Not Modified)  |
| POST   | 생성/행위 트리거 |         ✖ | 201(Created), 202       |
| PUT    | 전체 교체        |         ✔ | 200/204, 201(업서트 시) |
| PATCH  | 부분 수정        | ✖(일반적) | 200/204                 |
| DELETE | 삭제             |         ✔ | 204                     |

공통 오류: `400`(검증 실패), `401/403`(인증/인가), `404`(미존재), `409`(충돌), `422`(처리 불가), `429`(과다 요청), `5xx`(서버 오류).

---

## 5) REST의 장점·단점

### 장점

- **플랫폼 독립**: HTTP 표준만 따르면 어디서나 호출 가능.
- **느슨한 결합**: 클라·서버 역할 분리, 독립 배포 용이.
- **가시성**: cURL/Postman 등으로 테스트 쉬움, 캐시·프록시·CDN 친화.
- **진화 용이**: JSON과 같은 **자기서술적 포맷**은 **하위 호환**(새 필드는 무시) 설계에 유리.

### 단점

- **오버페치/언더페치**: 한 요청으로 딱 맞는 데이터 구성이 어려울 수 있음.
- **표현 과다(텍스트)**: JSON은 **가독성↑** 하지만 **용량/파싱 비용↑**.
- **행위 모델 한계**: 복잡한 도메인 행위를 **HTTP 메서드/URI**로만 표현이 애매할 수 있음.
- **단방향 요청/응답** 중심**(서버 푸시/스트리밍은 추가 기법 필요)**.

> 대안/보완: GraphQL(오버페치 해결), gRPC(바이너리·스트리밍), WebSocket/SSE(실시간).

---

## 6) 실무 베스트 프랙티스

### (1) 버전 관리

- URI 버전: `/v1/orders` (명시적·캐시 친화)
- 헤더 버전: `Accept: application/vnd.example.v2+json` (표현 협상 기반)

### (2) 페이지네이션·정렬·필터

- `page/size` 또는 커서 기반(`?cursor=...&limit=...`)
- `Content-Range`/`Link` 헤더로 페이지 정보 전달 고려

### (3) 캐싱·조건부 요청

- `ETag` + `If-None-Match`, `Last-Modified` + `If-Modified-Since`
- `Cache-Control: public, max-age=60`

### (4) 오류 응답 스키마 표준화

```json
{
  "error": "VALIDATION_ERROR",
  "message": "email is invalid",
  "details": [{ "field": "email", "reason": "format" }],
  "traceId": "c0a8018a..."
}
```

### (5) 보안

- TLS(HTTPS), OAuth2/OIDC, JWT/세션, **입력 검증**, **속도 제한(429)**, **CORS 정책**

### (6) 성능 최적화

- **압축**(gzip/br), **HTTP/2/3**(헤더 압축·멀티플렉싱), **CDN**
- **Partial Response**(필드 선택), **배치 엔드포인트**(대신 남용 주의)
- **HATEOAS/링크**로 탐색 가능성 제공(선택)

---

## 7) 간단 예시: 주문 도메인

```http
# 목록 조회(필터·페이지)
GET /orders?status=PAID&page=1&size=20
Accept: application/json

200 OK
ETag: "7d9c..."
Content-Type: application/json
[ { "id": 42, "status": "PAID" }, ... ]

# 생성
POST /orders
Content-Type: application/json
{ "customerId": 1001, "items": [{ "productId": 77, "qty": 2 }] }

201 Created
Location: /orders/42

# 조건부 조회(변경 없으면 304)
GET /orders/42
If-None-Match: "7d9c..."
```

---

## 8) 요약

- REST는 **자원 지향** + **HTTP 표준**을 활용하는 아키텍처 스타일입니다.
- **장점**: 단순·표준·캐시/프록시 친화, 느슨한 결합, 하위 호환 설계 용이.
- **단점**: 오버페치/언더페치, 페이로드/파싱 비용, 복잡 행위 표현의 제약.
- 실무에서는 **버전·캐시·오류 스키마·보안**을 표준화하고, 필요 시 **GraphQL/gRPC/실시간** 등으로 보완하십시오.
