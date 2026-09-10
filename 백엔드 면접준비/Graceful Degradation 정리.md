# Graceful Degradation 정리

## 핵심 요약

- 부분 장애가 전체 장애로 번지는 일을 줄인다.
- fallback 데이터가 오래되면 잘못된 의사결정을 만들 수 있다.
- 필수·선택 dependency를 API별로 표기한다.

## 개념 설명

Graceful Degradation은 dependency 일부가 실패해도 핵심 기능은 제한된 품질로 계속 제공하는 설계다.

추천, 통계, 부가 정보처럼 없어도 핵심 거래가 가능한 영역에는 timeout, fallback, stale data, feature flag를 준비한다.

## 예시

```text
product detail:
  price: required, fail closed
  recommendation: 150ms timeout, fallback empty
  review summary: stale cache allowed 10m
```

필수 데이터와 부가 데이터를 나누면 장애 중 어떤 응답을 줄지 API가 일관되게 결정할 수 있다.

## 면접 답변 예시

> Graceful degradation은 일부 dependency가 실패해도 핵심 기능은 제한된 품질로 계속 제공하는 설계입니다. API별로 price처럼 실패하면 닫아야 하는 필수 데이터와 recommendation처럼 비워도 되는 부가 데이터를 먼저 나누겠습니다. Stale cache를 fallback으로 쓸 때는 허용 시간과 사용자 표시 기준을 정해 오래된 값이 정상 값처럼 보이지 않게 해야 합니다. Fallback 사용률과 지속 시간을 정상 성공과 별도 지표로 남겨 조용한 부분 장애가 오래 숨지 않도록 합니다.

## 장점

- 사용자가 핵심 작업을 계속 완료할 수 있다.

## 단점

- 조용한 degradation은 장애 인지를 늦춘다.

## 주의사항 / 실무 팁

- degraded response에는 내부 지표와 사용자 표시 기준을 둔다.
