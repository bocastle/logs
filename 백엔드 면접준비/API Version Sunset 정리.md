# API Version Sunset 정리

## 핵심 요약

- 호출자가 정확한 종료 시각을 기계적으로 읽을 수 있다.
- Sunset 시각과 실제 route 차단 시각이 다르면 신뢰가 깨진다.
- Sunset 헤더와 상태 페이지의 종료 시각을 같게 맞춘다.

## 개념 설명

API Version Sunset은 공지된 시각 이후 특정 API 버전의 서비스를 종료하고 후속 호출에 명확한 종료 응답을 내리는 운영 상태다.

종료 전에 `Sunset` 헤더를 HTTP-date로 보내고, 종료 후에는 같은 route에서 `410 Gone`과 대체 버전 링크를 안정적으로 반환한다.

## 예시

```http
Sunset: Wed, 30 Sep 2026 23:59:59 GMT
Link: </docs/v2/orders>; rel="successor-version"

HTTP/1.1 410 Gone
{"code":"API_VERSION_SUNSET","successor":"/v2/orders"}
```

HTTP-date를 쓰면 타임존 해석을 통일할 수 있고, 410 Gone은 임시 장애와 영구 종료를 구분한다.

## 면접 답변 예시

> 대체 route와 migration 문서로 이동할 계약을 유지할 수 있다. 롤백 기준이 없으면 예상보다 많은 미전환 호출에 대응하기 어렵다. 비상 연장은 승인자, 새 종료일, 해당 client를 함께 기록한다.

## 장점

- 종료 후 재시도해도 복구되지 않는 이유를 명확히 알린다.

## 단점

- 410 Gone 대신 모호한 404를 보내면 client가 오류를 잘못 분류한다.

## 주의사항 / 실무 팁

- route 차단 후 410 Gone 비율과 client_id를 보존한다.
