# API Key Rotation 정리

## 핵심 요약

- API key rotation은 새 key를 발급해 client를 전환하고 이전 key 사용이 사라진 뒤 폐기하는 운영 절차다.
- Key ID와 secret을 분리하고 secret 원문은 발급 시 한 번만 보여 주며 server에는 검증용 hash만 저장한다.
- Scope, owner와 `last_used_at`을 key별로 추적하고 유출 시에는 migration window 없이 즉시 폐기할 경로를 둔다.

## 개념 설명

외부 연동 API key를 한 번에 바꾸면 배포 주기가 다른 client가 동시에 장애를 겪을 수 있다. 정상 rotation은 새 key를 만들고 일정 기간 두 key를 함께 검증한 뒤 이전 key의 사용이 멈췄는지 확인해 폐기한다.

Credential은 lookup용 key ID와 충분히 무작위인 secret으로 나눈다. Server는 secret 원문 대신 hash 또는 keyed hash를 저장하고 owner, scope, 생성·만료 시각과 마지막 사용 정보를 함께 관리한다. Key 값 자체는 query나 일반 access log에 남지 않게 한다.

## 예시

```text
key_id=ak_202607 active
key_id=ak_202606 deprecated until 2026-08-01
last_used_at tracked per client
```

이전 key의 `last_used_at`과 client 식별자를 보면 미전환 연동을 종료 전에 찾을 수 있다. 단, 유출이나 오남용이 확인된 key는 편의를 위한 dual 기간보다 즉시 폐기와 incident 대응이 우선이다.

## 면접 답변 예시

> API key rotation은 새 key를 발급하고 client가 전환할 기간을 둔 뒤 이전 key를 폐기하는 절차입니다. Key ID와 secret을 분리하고 원문은 발급 시 한 번만 보여 주며 server에는 검증용 hash만 저장합니다. Key별 owner, scope와 `last_used_at`을 추적해 미전환 client를 찾고 최소 권한을 적용합니다. 정상 rotation과 별개로 유출이 의심되면 즉시 폐기하고 대체 key를 발급할 비상 절차도 준비합니다.

## 장점

- 연동별 owner와 scope, 마지막 사용을 명확하게 관리할 수 있다.
- 정상 전환 기간을 제공하면서 장기 key의 유출 위험 기간을 줄일 수 있다.

## 단점

- Secret 원문을 저장하면 DB 유출 시 API credential이 바로 악용될 수 있다.
- Dual key 기간이 길수록 공격자가 사용할 수 있는 유효 credential 수도 늘어난다.

## 주의사항 / 실무 팁

- Dual key 기간, client별 마지막 사용과 폐기 날짜를 추적한다.
- Key가 query·log·trace에 남지 않는지 검사하고 고위험 scope에는 짧은 수명과 별도 승인 정책을 둔다.
