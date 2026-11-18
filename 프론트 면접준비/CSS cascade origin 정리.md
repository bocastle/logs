# CSS cascade origin 정리

## 핵심 요약

- 브라우저 기본값·사용자 스타일·제품 스타일의 관계를 설명한다.
- author `!important`가 모든 값을 이긴다고 생각하면 user important를 깨뜨린다.
- DevTools의 origin과 overridden reason을 확인한다.

## 개념 설명

CSS cascade origin은 선언이 user-agent, user, author 중 어디에서 왔는지와 normal·important·animation·transition 순서로 최종 값을 고르는 cascade의 출처 규칙이다.

normal 선언에서 author가 user와 UA보다 앞서지만 important 선언은 출처 순서가 뒤집힌다. cascade는 origin·importance·context·layer·specificity·scope·source order를 차례로 비교한다.

## 예시

```css
/* author stylesheet */
button { color: var(--button-fg); }
button.reset { color: revert; }
```

`revert`는 현재 origin의 값을 거두고 이전 origin의 cascade 결과로 돌아가므로 `initial`과 다르다.

## 면접 답변 예시

> CSS cascade는 먼저 origin과 importance를 비교하고, 같은 범위에서는 layer, specificity, scope proximity와 source order로 최종 선언을 고릅니다. Author `!important`가 항상 최상위인 것은 아니며 user important는 접근성 요구를 위해 author important보다 우선합니다. `revert`는 현재 origin의 선언을 거둬 이전 origin 결과로 돌아가고 `revert-layer`는 현재 cascade layer만 되돌린다는 차이도 설명하겠습니다. DevTools에서 origin과 overridden reason을 확인하고 transition 중 값이 정적 cascade 결과와 다르게 보이는 경우까지 분리해 진단합니다.

## 장점

- important override의 실제 우선순위를 정확히 파악한다.

## 단점

- origin과 layer를 specificity 문제로만 오판하기 쉽다.

## 주의사항 / 실무 팁

- user preference를 좀더 높은 author specificity로 덮지 않는다.
