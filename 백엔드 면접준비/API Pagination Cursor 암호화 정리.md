# API Pagination Cursor 암호화 정리

## 핵심 요약

- 내부 순번과 데이터 분포 노출을 줄인다.
- 키 회전 계획이 없으면 오래된 cursor 처리에서 장애가 난다.
- 현재 키와 이전 키를 검증 창 동안 함께 운영한다.

## 개념 설명

API Pagination Cursor 암호화는 cursor 안의 정렬 값과 내부 식별자가 호출자에게 보이지 않게 감추는 방식이다.

서버는 정렬 키와 필터 해시를 payload로 만든 뒤 내부 값을 숨겨야 하면 AEAD로 암호화한다. 서명만 적용하면 변조는 탐지할 수 있지만 payload의 기밀성은 제공하지 않는다.

## 예시

```text
payload = {last_id, sort_value, filter_hash, exp}
cursor = base64url(aead_encrypt(payload))
```

AEAD cursor는 내부 `last_id`를 숨기면서도 변조 여부를 한 번에 확인할 수 있다.

## 면접 답변 예시

> Pagination cursor에 내부 ID와 sort 값을 숨겨야 한다면 AEAD로 암호화해 기밀성과 변조 탐지를 함께 얻겠습니다. 서명만으로는 client가 payload를 읽는 것을 막지 못합니다. Payload에는 filter hash와 만료를 넣어 다른 query에 cursor를 재사용하지 못하게 하고 key ID로 현재·이전 key의 회전 기간을 운영합니다. 인증 실패 같은 암호 세부는 노출하지 않는 안정적인 invalid cursor 응답으로 처리하고 만료처럼 client가 새 탐색을 시작할 수 있는 상태만 명확히 안내합니다.

## 장점

- 클라이언트가 cursor 구조에 의존하지 않게 한다.

## 단점

- 토큰이 너무 커지면 URL 길이 제한에 걸릴 수 있다.

## 주의사항 / 실무 팁

- cursor 크기를 측정해 GET URL 한계를 넘지 않게 한다.
