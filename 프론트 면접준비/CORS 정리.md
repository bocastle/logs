# CORS 정리

## 핵심 요약

- CORS는 브라우저가 다른 출처의 리소스 요청을 허용할지 판단하는 보안 정책입니다.
- 출처는 **프로토콜, 도메인, 포트**가 모두 같아야 같은 출처로 봅니다.
- 프론트엔드에서 CORS 에러가 보이지만, 실제 허용 여부는 서버의 응답 헤더로 결정됩니다.
- 면접에서는 SOP와 CORS의 관계, preflight 요청, `Access-Control-Allow-Origin` 같은 헤더를 함께 설명하면 좋습니다.

## CORS란?

CORS는 Cross-Origin Resource Sharing의 약자입니다.  
말 그대로 다른 출처의 리소스를 공유할 수 있게 해주는 방식입니다.

웹에서는 기본적으로 **같은 출처 정책(Same-Origin Policy, SOP)** 이 있습니다.  
브라우저가 보안상 아무 사이트에서나 다른 사이트의 데이터를 마음대로 읽지 못하게 막는 정책입니다.

예를 들어 사용자가 은행 사이트에 로그인한 상태라고 해보겠습니다.  
악성 사이트가 몰래 은행 API를 호출해서 응답을 읽을 수 있다면 큰 문제가 됩니다. 그래서 브라우저는 기본적으로 다른 출처의 응답을 JavaScript가 읽지 못하게 제한합니다.

CORS는 이 제한을 무조건 없애는 것이 아니라, 서버가 허용한 경우에만 다른 출처 요청을 가능하게 해주는 장치입니다.

## 출처란?

출처는 다음 세 가지로 결정됩니다.

1. 프로토콜
2. 도메인
3. 포트

예를 들어 아래 두 주소는 서로 다른 출처입니다.

```txt
https://example.com
http://example.com
```

도메인은 같지만 프로토콜이 다르기 때문입니다.

아래도 다른 출처입니다.

```txt
https://example.com
https://api.example.com
```

서브도메인이 다르기 때문입니다.

포트가 달라도 다른 출처로 봅니다.

```txt
http://localhost:3000
http://localhost:8080
```

프론트 개발 중에 CORS를 자주 만나는 이유도 여기 있습니다.  
React 개발 서버는 `localhost:3000`, 백엔드 서버는 `localhost:8080`처럼 포트가 다른 경우가 많기 때문입니다.

## CORS 에러는 왜 프론트에서 보일까?

CORS 에러는 브라우저 콘솔에서 보이기 때문에 프론트엔드 문제처럼 느껴집니다.

하지만 실제로는 서버가 “이 출처에서 온 요청을 허용한다”는 응답 헤더를 내려줘야 해결됩니다.  
프론트 코드에서 아무리 요청을 잘 보내도, 서버 응답에 필요한 CORS 헤더가 없으면 브라우저가 응답을 막습니다.

대표적인 헤더는 다음과 같습니다.

```http
Access-Control-Allow-Origin: https://example.com
```

이 헤더는 `https://example.com` 출처에서 온 요청을 허용한다는 뜻입니다.

## 단순 요청과 preflight 요청

CORS 요청은 크게 단순 요청과 preflight 요청으로 나눠서 이해하면 좋습니다.

단순 요청은 브라우저가 바로 실제 요청을 보내는 경우입니다.  
예를 들어 조건을 만족하는 `GET`, `POST` 요청은 바로 서버로 갈 수 있습니다.

반면 어떤 요청은 실제 요청 전에 브라우저가 먼저 `OPTIONS` 요청을 보냅니다.  
이 요청을 preflight 요청이라고 합니다.

preflight는 쉽게 말해 “이런 메서드와 헤더로 요청을 보내도 되나요?”라고 서버에 미리 물어보는 과정입니다.

```http
OPTIONS /users HTTP/1.1
Origin: http://localhost:3000
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type, Authorization
```

서버가 허용한다고 응답하면 그때 실제 요청이 이어집니다.

## preflight가 발생하는 경우

preflight는 보통 이런 상황에서 발생합니다.

- `PUT`, `PATCH`, `DELETE` 같은 메서드를 사용할 때
- `Authorization` 같은 커스텀 헤더가 있을 때
- 특정 조건을 벗어난 `Content-Type`을 사용할 때
- credentials를 포함하는 요청을 보낼 때

그래서 로그인 토큰을 `Authorization` 헤더로 보내는 API에서는 preflight를 자주 보게 됩니다.

## 자주 보는 CORS 헤더

### Access-Control-Allow-Origin

요청을 허용할 출처를 지정합니다.

```http
Access-Control-Allow-Origin: https://example.com
```

개발 중에는 `*`를 쓰는 경우도 있지만, 인증 정보가 포함되는 요청에서는 주의해야 합니다.

### Access-Control-Allow-Methods

허용할 HTTP 메서드를 지정합니다.

```http
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

### Access-Control-Allow-Headers

허용할 요청 헤더를 지정합니다.

```http
Access-Control-Allow-Headers: Content-Type, Authorization
```

### Access-Control-Allow-Credentials

쿠키 같은 인증 정보를 포함한 요청을 허용할지 지정합니다.

```http
Access-Control-Allow-Credentials: true
```

이 값을 사용할 때는 `Access-Control-Allow-Origin: *`와 함께 쓸 수 없습니다.  
보안상 구체적인 Origin을 지정해야 합니다.

## credentials 옵션

프론트에서 쿠키를 포함해 요청하려면 `credentials` 설정이 필요합니다.

```js
fetch("https://api.example.com/me", {
  credentials: "include",
});
```

Axios라면 보통 이렇게 설정합니다.

```js
axios.get("https://api.example.com/me", {
  withCredentials: true,
});
```

하지만 프론트에서 이 옵션만 켠다고 끝나는 것은 아닙니다.  
서버도 `Access-Control-Allow-Credentials: true`를 내려줘야 하고, 허용 Origin도 `*`가 아니라 구체적인 출처여야 합니다.

## CORS와 보안

CORS는 서버를 보호하는 정책이라기보다, 브라우저가 응답을 JavaScript에 노출할지 제어하는 정책에 가깝습니다.

중요한 점은 CORS가 API 인증을 대신해주지 않는다는 것입니다.  
서버는 CORS 설정과 별개로 인증, 인가, CSRF 방어 등을 제대로 해야 합니다.

또 Postman이나 서버 간 요청에서는 CORS 에러가 나지 않을 수 있습니다.  
CORS는 브라우저가 적용하는 정책이기 때문입니다.

## 면접 답변 예시

CORS는 브라우저에서 다른 출처의 리소스를 요청할 때, 서버가 허용한 출처인지 확인하는 정책입니다.  
브라우저에는 기본적으로 같은 출처 정책이 있어서 프로토콜, 도메인, 포트가 다른 응답을 JavaScript가 마음대로 읽지 못하게 제한합니다. CORS는 서버가 `Access-Control-Allow-Origin` 같은 헤더로 허용한 경우에만 브라우저가 응답을 사용할 수 있게 해줍니다.

또 요청 메서드나 헤더가 단순 요청 조건을 벗어나면 브라우저가 실제 요청 전에 `OPTIONS` preflight 요청을 보내서 허용 여부를 먼저 확인합니다.  
프론트에서 CORS 에러가 보이지만, 보통은 서버에서 허용 Origin, 메서드, 헤더, credentials 설정을 맞춰줘야 해결됩니다.

## 장점

- 브라우저에서 다른 출처 리소스를 안전하게 공유할 수 있습니다.
- 서버가 허용할 Origin, 메서드, 헤더를 명시적으로 제어할 수 있습니다.
- 프론트와 백엔드가 다른 도메인에 있어도 필요한 API 통신을 할 수 있습니다.

## 단점

- 설정이 조금만 어긋나도 브라우저에서 요청이 막혀 디버깅이 헷갈릴 수 있습니다.
- 프론트 콘솔에 에러가 보여도 실제 수정은 서버 설정에서 해야 하는 경우가 많습니다.
- credentials 요청에서는 `*`를 쓸 수 없어 환경별 Origin 관리가 필요합니다.

## 주의사항 / 실무 팁

- CORS 에러가 나면 먼저 요청 Origin과 서버 응답의 CORS 헤더를 같이 확인하세요.
- 쿠키 인증을 쓴다면 프론트의 `credentials: "include"`와 서버의 `Access-Control-Allow-Credentials: true`가 함께 필요합니다.
- 운영 환경에서 `Access-Control-Allow-Origin: *`를 습관적으로 쓰는 것은 피하는 것이 좋습니다.
- Postman에서는 되는데 브라우저에서 안 된다면 CORS 문제일 가능성이 높습니다.
- CORS는 인증/인가를 대체하지 않으므로 서버 보안 로직은 별도로 확실히 구현해야 합니다.
