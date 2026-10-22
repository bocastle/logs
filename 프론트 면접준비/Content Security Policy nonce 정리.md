# Content Security Policy nonce 정리

## 핵심 요약

- 허용된 inline script만 실행되도록 제한할 수 있다.
- nonce가 로그나 DOM에 노출되면 우회 가능성이 생긴다.
- nonce는 요청마다 CSPRNG로 생성한다.

## 개념 설명

Content Security Policy nonce는 서버가 응답마다 예측 불가능한 nonce를 만들고 허용할 inline script에만 붙여 XSS 실행면을 줄이는 정책이다.

`script-src 'nonce-...'`와 script 태그의 `nonce` 값이 일치해야 실행되고, nonce는 응답마다 새로 생성되어야 한다.

## 예시

```http
Content-Security-Policy: script-src 'nonce-r4nd0m' 'strict-dynamic'; object-src 'none'
```

```html
<script nonce="r4nd0m">bootstrap()</script>
```

정책과 태그 nonce가 맞는 inline script만 허용한다. 고정 nonce나 재사용 nonce는 공격자가 재사용할 수 있어 위험하다.

## 면접 답변 예시

> CSP nonce는 server가 응답마다 예측 불가능한 값을 만들고 허용한 script element에만 붙여 arbitrary inline script 실행을 줄이는 방식입니다. 같은 nonce를 HTML cache가 여러 응답에 재사용하지 않게 하고 template에 user input을 nonce 위치나 script body로 섞지 않겠습니다. `strict-dynamic`을 쓰면 nonce를 가진 신뢰 script가 만든 script의 허용 관계가 바뀌므로 target browser와 fallback source 정책을 함께 설계해야 합니다. Report-Only로 legacy inline handler를 찾은 뒤 `object-src 'none'`, `base-uri` 같은 방어와 함께 적용하며 TLS와 sanitization도 별도로 유지합니다.

## 장점

- hash보다 동적으로 생성되는 bootstrap script에 적용하기 쉽다.

## 단점

- 템플릿 캐시가 nonce를 재사용하면 정책 의미가 깨진다.

## 주의사항 / 실무 팁

- HTML 캐시와 CDN이 nonce를 재사용하지 않는지 확인한다.
