# Web Crypto API 정리

## 핵심 요약

- Web Crypto API는 browser가 제공하는 난수, hash, 서명과 암복호화 primitive를 비동기로 사용하게 한다.
- API를 사용한다고 안전한 protocol이 자동으로 만들어지지는 않으므로 algorithm·nonce·key 수명을 검토된 설계에서 가져온다.
- Client에 전달한 key는 `extractable: false`여도 page를 제어하는 사용자와 XSS로부터 완전히 숨길 수 없다.

## 개념 설명

Web Crypto API는 `crypto.getRandomValues()`와 `SubtleCrypto`를 통해 browser의 암호 구현을 사용하게 한다. 직접 난수 생성기나 암호 algorithm을 구현하는 것보다 낫지만 key 용도와 protocol 설계는 application 책임이다.

`CryptoKey`를 non-extractable로 만들면 API를 통해 raw key를 다시 export하는 것을 막을 수 있다. 그러나 같은 origin에서 실행되는 script는 허용된 sign·decrypt operation을 호출할 수 있고, 처음 raw secret을 import했다면 그 값은 이미 JavaScript memory에 있었다. Browser에 shared HMAC secret을 배포해 server가 client를 신뢰하는 방식은 안전한 인증 경계가 아니다.

## 예시

```ts
const key = await crypto.subtle.importKey("raw", secret, "HMAC", false, ["sign"]);
const sig = await crypto.subtle.sign("HMAC", key, payload);
const nonce = crypto.getRandomValues(new Uint8Array(12));
```

예시는 이미 안전하게 전달된 secret으로 payload integrity를 계산하는 primitive일 뿐 전체 인증 protocol은 아니다. HMAC nonce와 AEAD nonce의 길이·재사용 조건을 혼동하지 않고 key마다 허용 operation을 최소화한다.

## 면접 답변 예시

> Web Crypto API는 browser가 제공하는 난수, hash, 서명과 암호화 primitive를 사용하는 방법입니다. 직접 암호 algorithm을 만들지 않고 검토된 protocol의 algorithm과 nonce 규칙을 그대로 적용하겠습니다. `extractable: false`는 raw key export를 막지만 같은 page의 script가 key operation을 사용하는 것까지 막지는 못합니다. Text encoding과 byte 직렬화를 server와 맞추고 서명·암호화 key의 용도와 수명을 분리합니다.

## 장점

- Browser의 검증된 암호 구현과 안전한 난수원을 사용할 수 있다.
- Key의 export 가능 여부와 허용 operation을 API 수준에서 제한할 수 있다.

## 단점

- Client에 있는 key는 사용자와 같은 origin의 악성 script로부터 완전히 숨길 수 없다.
- Algorithm과 nonce 사용을 잘못 선택하면 표준 API를 사용해도 보안이 깨진다.

## 주의사항 / 실무 팁

- `TextEncoder`, ArrayBuffer와 signature encoding 규칙을 server와 test vector로 맞춘다.
- Key usage, origin의 CSP·XSS 방어와 rotation·폐기 경계를 함께 설계한다.
