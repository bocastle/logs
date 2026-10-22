# Object Storage Presigned URL 정리

## 핵심 요약

- 대용량 파일 트래픽을 애플리케이션 서버에서 분리할 수 있다.
- URL이 유출되면 만료 전까지 같은 권한이 사용될 수 있다.
- presigned URL 만료를 짧게 두고 object key를 예측 어렵게 만든다.

## 개념 설명

Object Storage Presigned URL은 서버가 짧은 시간 동안 특정 객체 작업을 허용하는 서명 URL을 발급해 클라이언트가 직접 업로드나 다운로드하게 하는 방식이다.

서버는 object key, method, content type, 만료 시간을 서명하고, 클라이언트는 해당 조건으로 S3 같은 object storage에 직접 요청한다.

## 예시

```text
POST /uploads -> presigned PUT url expires=5m
client PUT object with Content-Type=image/png
server verifies callback or HEAD before publish
```

presigned URL은 권한 위임이다. 만료 시간, object key 범위, 업로드 후 검증 상태를 꼭 나눠야 한다.

## 면접 답변 예시

> Presigned URL은 application server가 짧은 시간 동안 특정 object key와 method의 권한을 위임해 client가 storage에 직접 upload하도록 하는 방식입니다. URL 자체가 만료 전 bearer credential이므로 key 범위와 만료를 최소화하고 log나 analytics에 남기지 않겠습니다. Storage가 지원하는 signed header나 policy로 content type, size와 checksum 조건을 묶되 client 값만 신뢰하지 않습니다. Upload 성공과 공개 상태를 분리해 server가 HEAD, 실제 file type과 virus scan을 확인한 뒤 publish합니다.

## 장점

- 짧은 만료와 제한된 object key로 권한 범위를 줄인다.

## 단점

- content type이나 크기 제한 검증이 약하면 원치 않는 객체가 올라간다.

## 주의사항 / 실무 팁

- 업로드 후 HEAD와 virus scan을 거쳐 공개 상태로 바꾼다.
