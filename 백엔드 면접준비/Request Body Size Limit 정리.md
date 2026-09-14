# Request Body Size Limit 정리

## 핵심 요약

- 큰 요청이 서버 메모리를 점유하는 시간을 줄인다.
- 프록시 한도와 앱 한도가 다르면 응답 형식이 흔들린다.
- 경로별 최대 크기와 예외 API를 문서화한다.

## 개념 설명

Request Body Size Limit은 API가 받을 수 있는 payload 크기를 제한해 메모리, 디스크, upstream 비용을 보호하는 정책이다.

gateway와 애플리케이션의 최대 크기를 맞추고, 초과 요청은 body를 끝까지 읽기 전에 413으로 거절한다.

## 예시

```http
HTTP/1.1 413 Payload Too Large
Content-Type: application/problem+json

{"code":"REQUEST_BODY_TOO_LARGE","max_bytes":1048576}
```

413 응답에 `max_bytes`를 넣으면 클라이언트가 압축, 분할 업로드, 다른 API 선택을 판단할 수 있다.

## 면접 답변 예시

> Request body size limit은 큰 payload가 gateway, application memory와 downstream 비용을 과도하게 점유하지 않게 하는 방어선입니다. 가능한 한 body 전체를 buffering하기 전에 한도를 적용하고 초과하면 일관된 413 응답을 주겠습니다. Wire 크기만 제한하면 작은 압축 요청이 해제 후 크게 팽창할 수 있어 압축 전송량과 decompressed 크기 모두에 상한이 필요합니다. 일반 JSON과 streaming upload는 경로별 정책을 나누고 413 발생량과 client 사용 패턴을 보고 정상 요청을 잘못 막는지도 확인합니다.

## 장점

- gateway와 앱의 실패 응답을 통일할 수 있다.

## 단점

- 스트리밍 업로드를 같은 한도로 막으면 정상 사용성이 떨어진다.

## 주의사항 / 실무 팁

- 압축 전후 크기 제한을 모두 검토한다.
