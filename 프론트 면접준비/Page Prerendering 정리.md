# Page Prerendering 정리

## 핵심 요약

- 다음 page의 표시 대기를 큰 폭으로 줄일 수 있다.
- 잘못된 예측은 네트워크·CPU·서버 부하를 낭비한다.
- 부작용 없는 읽기 page와 확률 높은 funnel부터 적용한다.

## 개념 설명

Page Prerendering은 사용자가 다음에 이동할 가능성이 높은 page를 숨겨진 상태에서 미리 navigation·render한 뒤 이동 시 activation하는 전환 최적화다.

Speculation Rules에 prerender URL과 eagerness를 정의하면 브라우저가 적격성·resource 상태를 판단해 미리 page를 준비한다. prerender 중 부작용·인증·실시간 연결을 제한하고 activation 후 완료한다.

## 예시

```html
<script type="speculationrules">
{"prerender":[{"source":"list","urls":["/checkout"],"eagerness":"moderate"}]}
</script>
```

상품 요약에서 다음 단계 checkout을 moderate prerender 후보로 두되 결제 부작용은 activation 전에 실행하지 않는다.

## 면접 답변 예시

> 예측 가능한 funnel 단계에 성능 예산을 집중한다. 개인화·인증 상태가 activation 시점에 바뀌었을 수 있다. `activationStart`, prerender hit rate, waste byte를 함께 측정한다.

## 장점

- HTML·CSS·JS·data 준비를 navigation 전으로 앞당긴다.

## 단점

- GET에 부작용이 있는 page는 prerender 중 문제를 만든다.

## 주의사항 / 실무 팁

- Save-Data·battery·memory·network 조건에서 후보를 줄인다.
