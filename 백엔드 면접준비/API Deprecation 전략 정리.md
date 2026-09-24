# API Deprecation 전략 정리

## 핵심 요약

- client별 전환 장애를 사용 데이터로 우선순위화할 수 있다.
- client_id 식별이 없으면 남은 호출의 소유자를 찾기 어렵다.
- version과 client_id를 낮은 cardinality 지표와 구조화 로그에 남긴다.

## 개념 설명

API Deprecation 전략은 사용중지 예고를 넘어 실제 client 전환을 usage telemetry, 지원, 단계적 제한으로 완료하는 migration 프로그램이다.

버전별 `client_id` usage telemetry로 소유자를 찾고, 신규 가입 freeze, warning, 테스트 차단, 전체 차단을 순서대로 진행한다.

## 예시

```text
usage telemetry: v1 requests by client_id
T-90d: announce and migration support
T-60d: new-client freeze
T-14d: scheduled test rejection
T-0: disable v1
```

freeze는 기존 client를 바로 끊지 않으면서 오래된 버전의 새 의존성이 늘어나는 것을 먼저 막는 단계다.

## 면접 답변 예시

> 지원·영업·개발이 같은 전환 대상 목록을 사용할 수 있다. freeze 예외를 무제한 허용하면 종료 대상이 계속 늘어난다. 테스트 차단 일정을 제공해 client의 실제 fallback을 확인한다.

## 장점

- 단계별 정지 조건이 있어 전체 차단 위험을 줄인다.

## 단점

- usage telemetry에 내부 배치를 빼먹으면 차단 당일 장애가 난다.

## 주의사항 / 실무 팁

- 단계별 종료 조건에 잔여 호출량과 핵심 client 전환률을 넣는다.
