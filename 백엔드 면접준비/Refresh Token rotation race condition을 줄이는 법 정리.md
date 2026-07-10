# Refresh Token rotation race condition을 줄이는 법 정리

## 핵심 요약

- 동시 refresh에서 최신 token이 두 개 발급되는 것을 막는다.
- grace window가 길면 탈취 token의 재사용 창도 넓어진다.
- grace window는 실제 client 재전송 시간 분포로 정한다.

## 개념 설명

Refresh Token rotation race condition은 한 token으로 여러 refresh 요청이 거의 동시에 들어와 정상 요청이 재사용으로 오탐되거나 두 개의 최신 token이 생기는 경쟁이다.

서버는 `family_id`와 version을 compare-and-swap으로 한 번만 갱신하고, 짧은 grace window에서 같은 client의 중복 요청에는 첫 번째 rotation 결과를 재사용한다.

## 예시

```text
UPDATE token_family
SET version=8, result_ref='r8'
WHERE family_id='f1' AND version=7  # compare-and-swap
duplicate within grace window -> return result_ref r8
```

compare-and-swap가 실패한 요청을 grace window 안의 정상 중복과 창 밖의 reuse로 나누어야 보안과 사용성을 같이 지킬 수 있다.

## 면접 답변 예시

> family_id 단위로 경쟁과 reuse 이벤트를 추적할 수 있다. 저장소가 compare-and-swap을 보장하지 않으면 경쟁이 그대로 남는다. 경쟁 통합 테스트로 한 rotation만 성공하는지 확인한다.

## 장점

- 정상적인 모바일 재전송을 공격으로 오탐하는 비율을 줄인다.

## 단점

- rotation 결과를 안전하게 재사용하지 못하면 중복 요청에 새 token을 노출할 수 있다.

## 주의사항 / 실무 팁

- family_id, version, client 지문을 token 원문 없이 로그에 남긴다.
