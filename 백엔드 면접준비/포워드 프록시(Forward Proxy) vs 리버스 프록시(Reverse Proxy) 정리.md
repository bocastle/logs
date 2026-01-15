# 포워드 프록시(Forward Proxy) vs 리버스 프록시(Reverse Proxy) 정리

> 요약: **포워드 프록시**는 **클라이언트 앞**에서 외부로 나가는 트래픽을 대리하고, **리버스 프록시**는 **서버 앞**에서 외부에서 들어오는 트래픽을 대리합니다. 전자는 **클라이언트 보호/정책/익명성**, 후자는 **서버 보호/부하 분산/SSL 종료**가 핵심 목적입니다.

---

## 1) 한눈에 비교

| 항목            | 포워드 프록시(Forward Proxy)                    | 리버스 프록시(Reverse Proxy)                      |
| --------------- | ----------------------------------------------- | ------------------------------------------------- |
| 배치 위치       | 클라이언트 측(내부망 출구)                      | 서버 측(퍼블릭 엔드포인트)                        |
| 기본 목적       | 익명화, 접근 제어, 캐싱, DLP, 감사              | 로드 밸런싱, 보안 차폐, SSL 종료, 캐싱/압축       |
| IP 관점         | **서버는 프록시의 IP만 봄**(클라이언트 IP 은닉) | **클라이언트는 프록시의 IP만 봄**(백엔드 IP 은닉) |
| 트래픽 흐름     | Client → **Forward Proxy** → Internet(Server)   | Client → **Reverse Proxy** → Backend Servers      |
| 캐싱 초점       | 외부 컨텐츠 재사용(아웃바운드 절감)             | 내부 정적/동적 응답 가속(인바운드 절감)           |
| 인증/정책       | 사용자/그룹 기반 정책, URL 필터링               | WAF/봇 차단, 속도 제한, 라우팅 규칙               |
| 대표 소프트웨어 | Squid, Blue Coat, CNTLM 등                      | Nginx, HAProxy, Envoy, Apache, Traefik            |
| 클라 설정 필요  | 보통 필요(브라우저/OS/에이전트에 프록시 지정)   | 필요 없음(프록시는 서버 쪽 인프라)                |

---

## 2) 경로/흐름(개념 다이어그램)

### 2.1 Forward Proxy

```
(Private LAN)            (Internet)
┌─────────┐  HTTP/HTTPS   ┌────────────┐
│ Client  │──────────────>│  Website   │
└────┬────┘               └────────────┘
     │
     │ via proxy
     ▼
┌──────────────┐
│ ForwardProxy │  ← 접근 제어/캐싱/익명화
└──────────────┘
```

### 2.2 Reverse Proxy

```
(Internet)                  (Private DC/VPC)
┌─────────┐ HTTP/HTTPS  ┌──────────────┐  HTTP/gRPC ...
│ Client  │────────────>│ ReverseProxy │────────────> [App1]
└─────────┘             └──────┬───────┘              [App2]
                                └──────────────> [Static/Media]
```

---

## 3) 포워드 프록시 — 주요 기능/사례

1. **익명성/프라이버시**: 외부에서는 클라이언트의 실제 IP가 보이지 않습니다.
2. **접근 통제/정책**: URL 카테고리/도메인 화이트·블랙리스트, 업무 외 사이트 차단.
3. **캐싱**: 반복 다운로드 절감(패키지 레지스트리, 대용량 업데이트, 이미지 등).
4. **감사/포렌식**: 감사 로그/사용자 식별, DLP(Data Loss Prevention) 연계.
5. **인증/프록시 체인**: NTLM/Basic/SSO, 상위 프록시(업스트림) 연결.

> 주의: HTTPS에 대해서는 CONNECT 터널 사용 시 **가시성 제약**이 있으며, SSL 인터셉트(기업 루트 CA 삽입)는 보안/윤리/합규 제약을 동반합니다.

---

## 4) 리버스 프록시 — 주요 기능/사례

1. **로드 밸런싱**: 라운드로빈/가중치/최소연결/지연 기반, 헬스체크/회로 차단.
2. **보안 차폐**: 백엔드 IP 숨김, 레이트 리밋/Geo 블록, WAF/봇 차단, DDoS 보호.
3. **SSL/TLS 종료**: 인증서 관리 중앙화, HTTP/2·HTTP/3, HSTS/ALPN.
4. **캐싱/압축/리라이팅**: 정적 자산 캐시, 브로틀링, 이미지 최적화, 헤더 관리(CORS 등).
5. **스마트 라우팅**: 경로/호스트 기반, A/B 테스트, 카나리 릴리스, 다중 리전 장애 조치.

---

## 5) 공통점과 차이의 핵심

- **공통**: “중개자”로서 캐싱/정책/로깅/관측 가능, 네트워크 경계에 위치.
- **결정적 차이**: **누구를 대신하느냐** — 클라이언트(Forward) vs 서버(Reverse). 이 차이가 **정책 주체/보안 책임/구성 위치**를 완전히 바꿉니다.

---

## 6) 구성 예시

### 6.1 Reverse Proxy — Nginx(기본)

```nginx
server {
  listen 443 ssl http2;
  server_name api.example.com;

  ssl_certificate     /etc/ssl/certs/fullchain.pem;
  ssl_certificate_key /etc/ssl/private/privkey.pem;

  # 보안 헤더/압축 등
  add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
  gzip on;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://app_backend;   # upstream 블록 정의 필요
  }
}
upstream app_backend {
  server 10.0.1.10:8080 max_fails=3 fail_timeout=10s;
  server 10.0.1.11:8080;
}
```

### 6.2 Forward Proxy — Squid(개요)

```conf
# /etc/squid/squid.conf
http_port 3128
acl localnet src 10.0.0.0/8
http_access allow localnet
http_access deny all
cache_mem 256 MB
maximum_object_size 32 MB
```

> 브라우저/OS에 `http://proxy.local:3128` 을 프록시로 지정해야 아웃바운드가 프록시를 경유합니다.

---

## 7) 선택 가이드

- **사내 단말의 아웃바운드 통제/감사/캐싱**이 필요하면 → **포워드 프록시**.
- **공개 서비스의 트래픽 분산/보안/TLS 관리**가 필요하면 → **리버스 프록시**.
- 대규모 서비스는 보통 **두 종류 모두**를 서로 다른 경계에서 운용합니다.

---

## 8) 운영 시 주의/베스트 프랙티스

- **TLS/인증서 수명 관리**: 자동 갱신(Let’s Encrypt)·보안 정책(HSTS/OCSP).
- **관측**: 접근 로그/지표(status code, latency, upstream health), 대시보드·경보.
- **캐시 무효화 규칙**: Key·TTL·변조 감지(ETag/Cache-Control) 설계.
- **보안 헤더**: CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy.
- **백프레셔/레이트 리밋**: 스파이크/오버로드 완화, 서킷 브레이커와 연계.
- **IPv6/HTTP/2/3**: 최신 프로토콜 활성화로 성능·호환성 개선.

---

## 9) 요약

- **Forward Proxy**: 클라이언트를 대신해 외부로 나감 — **익명화/정책/캐싱/감사**.
- **Reverse Proxy**: 서버를 대신해 외부 요청을 수신 — **로드 밸런싱/보안 차폐/SSL 종료/가속**.
- 맥락에 맞춰 두 유형을 적절히 배치하면 **보안과 성능, 운영 편의**를 동시에 달성할 수 있습니다.
