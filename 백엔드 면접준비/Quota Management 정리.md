# Quota Management 정리

## 핵심 요약

- 요금제와 인프라 보호 정책을 같은 숫자로 관리할 수 있다.
- metering 지연이 크면 한도를 넘어선 뒤에야 차단된다.
- 사용량 원장과 차단 결정을 감사 가능하게 남긴다.

## 개념 설명

Quota Management는 기간별 사용량, 용량, 비용 한도를 계정이나 tenant 단위로 배분하고 집행하는 운영 체계다.

metering은 실제 사용량을 기록하고, enforcement는 soft limit 경고와 hard limit 차단을 API 응답으로 일관되게 표현한다.

## 예시

```text
monthly_export_rows_used += rows
soft_limit: emit warning header
hard_limit: 403 QUOTA_EXCEEDED
```

soft limit과 hard limit을 나누면 고객에게 조정 시간을 주면서 인프라 비용 폭주를 막을 수 있다.

## 면접 답변 예시

> Quota management는 tenant별 기간 사용량과 용량·비용 한도를 측정하고 경고나 차단으로 집행하는 체계입니다. Metering과 enforcement가 지연되면 실제 한도를 넘긴 뒤 차단될 수 있어 사용량 원장과 결정 시각을 감사 가능하게 남기겠습니다. Soft limit에서는 미리 알리고 hard limit에서는 안정적인 error code, reset 시각과 해결 방법을 제공해 일반 장애와 구분합니다. 관리자 예외 증설은 actor, 이유와 만료 시각을 기록해 영구 우회가 되지 않게 합니다.

## 장점

- 사용량 알림으로 갑작스러운 차단을 줄인다.

## 단점

- quota 오류가 모호하면 사용자는 장애로 오해한다.

## 주의사항 / 실무 팁

- 응답 코드와 헤더로 남은 quota를 알려 준다.
