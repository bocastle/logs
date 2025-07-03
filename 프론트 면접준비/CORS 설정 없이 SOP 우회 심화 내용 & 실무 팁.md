# CORS 설정 없이 SOP 우회 — 심화 내용 & 실무 팁

## 1. JSONP (JSON with Padding) — 구식이지만 이해 필요

- `<script>` 태그는 SOP 제한 없음 → 외부 서버에서 콜백 함수로 감싼 JS를 전달
- GET 요청만 가능, 보안 취약성 있음 → 현대에는 거의 사용하지 않음

```html
<!-- 클라이언트에서 콜백 함수 정의 -->
<script>
  function handleResponse(data) {
    console.log("서버 응답:", data);
  }
</script>

<!-- 외부 서버 요청: 콜백 함수 이름을 쿼리 파라미터로 전달 -->
<script src="https://external-server.com/api?callback=handleResponse"></script>
```

## 2. 서버 측 프록시 구현 시 고려사항

- 인증 헤더/쿠키 전달 문제
- 응답 캐싱으로 성능 향상 가능
- 에러 핸들링 로직 필요

#### 서버 측 프록시 구현 (Node.js Express 예시)

```js
const express = require("express");
const axios = require("axios");
const app = express();

app.get("/api/proxy", async (req, res) => {
  try {
    const externalResponse = await axios.get(
      "https://external-server.com/data",
      {
        headers: {
          Authorization: `Bearer ${process.env.API_TOKEN}`,
        },
      }
    );
    res.json(externalResponse.data);
  } catch (error) {
    res.status(500).json({ error: "Proxy error" });
  }
});

app.listen(3000, () => console.log("Proxy server running on port 3000"));
```

## 3. CORS 프리플라이트(Preflight)와 프록시의 관계

- 브라우저가 OPTIONS 요청 보내는 부분을 프록시가 처리해야 함

## 4. 서버 간 요청과 보안 취약점 (SSRF) 주의

- 프록시가 외부 URL 임의 요청 가능 시 SSRF 취약점 가능성
- 내부망 접근 제한, 화이트리스트 필터링 필요

```js
const allowedHosts = ["external-server.com", "api.trusted.com"];

function isValidHost(url) {
  const host = new URL(url).hostname;
  return allowedHosts.includes(host);
}

app.get("/api/proxy", async (req, res) => {
  const targetUrl = req.query.url;
  if (!isValidHost(targetUrl)) {
    return res.status(403).send("Forbidden");
  }
  // ...
});
```

## 5. 대체 기술 — CORS 우회 확장

- 브라우저 확장 프로그램 (개발용)
- 공개 프록시 서비스 이용 가능하나 보안/속도 문제 주의

## 6. WebSocket, SSE 등 실시간 통신 시 CORS/SOP 고려

- 실시간 통신에도 별도 설정과 프록시 조율 필요
