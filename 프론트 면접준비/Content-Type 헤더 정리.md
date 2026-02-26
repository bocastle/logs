# Content-Type 헤더 정리

## 핵심 요약

- `Content-Type`은 HTTP 요청/응답에서 **전송되는 본문(body) 데이터의 형식**을 나타내는 헤더입니다.
- 올바르게 지정하지 않으면 서버/클라이언트가 데이터를 잘못 해석하거나, 경우에 따라 `415 Unsupported Media Type` 같은 오류가 발생할 수 있습니다.
- `Accept`는 **응답으로 받고 싶은 형식**, `Content-Type`은 **지금 보내는 데이터의 형식**을 지정합니다.

## Content-Type 헤더란?

`Content-Type`은 HTTP 요청과 응답에서 전송되는 데이터의 타입을 명시하는 헤더입니다. 서버와 클라이언트가 데이터를 주고받을 때, 올바르게 해석할 수 있도록 하기 위해 사용합니다.

예를 들어, JSON 데이터를 전송할 경우 `Content-Type: application/json`을 사용하면 서버는 해당 데이터가 JSON 형식이라는 것을 알고 적절한 방식으로 해석할 수 있습니다. 혹은 서버가 클라이언트에게 HTML을 응답할 때 `Content-Type: text/html`을 지정하면 브라우저는 이를 HTML로 렌더링할 수 있습니다.

`Content-Type` 헤더는 **MIME 타입**을 기반으로 하며, `[type]/[subtype]` 형식으로 구성됩니다. 예를 들어, JSON 데이터는 `application/json`, HTML 문서는 `text/html`를 사용합니다. 만약 파일을 업로드하는 경우에는 `multipart/form-data`를 사용합니다.

`Content-Type` 헤더를 정확하게 지정하지 않으면, 클라이언트나 서버에서 데이터를 올바르게 해석하지 못할 수 있습니다. 예를 들어, JSON 데이터를 전송하면서 `Content-Type: application/x-www-form-urlencoded`로 설정하면 서버는 데이터를 잘못된 방식으로 처리하거나, `415 Unsupported Media Type`를 반환할 수 있습니다.

## Content-Type과 Accept 헤더의 차이

`Content-Type` 헤더는 **전송되는 데이터의 타입**을 지정하는 반면, `Accept` 헤더는 **응답으로 받고자 하는 데이터의 타입**을 지정합니다.

예를 들어, 클라이언트가 JSON 응답을 원할 경우 `Accept: application/json`을 설정하면, 서버는 가능하면 JSON 형식으로 응답을 반환하게 됩니다.

## 예시(헤더/요청)

```http
# JSON 요청을 보낼 때
Content-Type: application/json
Accept: application/json

# HTML 응답을 받을 때(브라우저/서버)
Content-Type: text/html

# 파일 업로드(폼 데이터)
Content-Type: multipart/form-data
```

## 장점

- 서버/클라이언트가 본문 데이터를 **정확하게 파싱/처리**할 수 있습니다.
- 응답 렌더링(예: `text/html`)이나 API 파싱(예: `application/json`)의 **일관성**을 높입니다.

## 단점

- 잘못 지정하면 파싱 오류, 서버 처리 실패, `415 Unsupported Media Type` 등 **호환성 문제가 즉시 발생**할 수 있습니다.

## 주의사항 및 실무 팁

- API 요청 시 `Content-Type`과 `Accept`를 혼동하지 않도록 역할을 구분합니다.
- 서버가 특정 `Content-Type`만 허용하는 경우(예: JSON만 허용) 클라이언트에서 정확히 맞춰야 합니다.
- 파일 업로드는 `multipart/form-data`를 쓰며, 라이브러리/브라우저가 boundary를 자동으로 설정하는 경우가 많으니 직접 헤더를 고정하기 전에 동작 방식을 확인합니다.
