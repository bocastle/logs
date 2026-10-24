# API Request Signing 정리

## 핵심 요약

- body 변조와 중간자 요청 재사용을 줄인다.
- canonicalization 문서가 모호하면 연동 실패가 많아진다.
- SDK와 서버가 같은 테스트 벡터로 signature를 검증한다.

## 개념 설명

API Request Signing은 요청 method, path, body hash, timestamp를 비밀키로 서명해 호출자와 요청 무결성을 확인하는 방식이다.

서버는 canonical request를 같은 규칙으로 만들고 signature, key id, timestamp 허용 범위, nonce 재사용 여부를 검증한다.

## 예시

```text
canonical = method + "\n" + path + "\n" + sha256(body) + "\n" + timestamp
signature = HMAC-SHA256(secret, canonical)
```

canonical 규칙이 양쪽에서 같아야 공백, 쿼리 순서, 헤더 대소문자 차이로 정상 요청이 실패하지 않는다.

## 면접 답변 예시

> API request signing은 method, canonical path·query, selected header, body hash와 timestamp를 서명해 caller와 request integrity를 확인하는 방식입니다. Canonicalization 규칙이 공백, encoding과 header case까지 명확해야 SDK와 server가 같은 byte를 서명합니다. Timestamp 허용 범위와 nonce replay store를 함께 검증하고 key ID로 rotation을 지원하겠습니다. HMAC secret은 mobile이나 browser app에서 안전하게 숨길 수 없어 server-to-server에 적합하며 TLS와 authorization을 대체하지 않습니다.

## 장점

- API key 단독 인증보다 무결성 보장이 강하다.

## 단점

- timestamp 검증이 없으면 replay에 취약하다.

## 주의사항 / 실무 팁

- key id별 회전과 폐기 절차를 둔다.
