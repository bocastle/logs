# 로그 마스킹 정책을 API 경계에 적용하는 법 정리

## 핵심 요약

- 로그 분석과 개인정보 보호를 함께 유지할 수 있다.
- denylist 방식은 새 민감 필드를 놓치기 쉽다.
- 가능하면 allowlist 기반 구조화 로그를 사용한다.

## 개념 설명

API 경계의 로그 마스킹은 요청·응답·오류 로그에 민감정보가 저장되기 전에 필드 단위로 가리는 정책이다.

schema나 allowlist 기반으로 password, token, 주민번호, 카드번호 같은 필드를 redaction하고, 원문이 필요한 경우는 별도 보안 저장소로 분리한다.

## 예시

```text
request.email = b***@example.com
request.authorization = [REDACTED]
response.card_number = [MASKED_LAST4:4242]
```

token과 Authorization 헤더는 일부 노출도 재사용 위험이 있으므로 `[REDACTED]`처럼 완전히 제거하는 편이 안전하다.

## 면접 답변 예시

> 감사 시 어떤 필드가 저장되는지 설명하기 쉽다. 과도한 마스킹은 장애 분석에 필요한 키까지 없앨 수 있다. 마스킹 정책 변경은 샘플 로그로 회귀 테스트한다.

## 장점

- API gateway와 앱 로그의 정책 차이를 줄인다.

## 단점

- 마스킹 전 예외 로그가 원문을 남길 수 있다.

## 주의사항 / 실무 팁

- 예외 처리와 access log 모두 같은 redaction 함수를 거치게 한다.
