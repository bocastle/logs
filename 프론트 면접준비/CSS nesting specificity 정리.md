# CSS nesting specificity 정리

## 핵심 요약

- 관련 selector를 컴포넌트 경계 안에 모은다.
- ID와 class를 같은 parent list에 묶으면 specificity가 필요 이상으로 커진다.
- parent selector list의 specificity를 함께 검토한다.

## 개념 설명

CSS nesting specificity는 nested selector list가 펼쳐질 때 일반 selector specificity와 같이 cascade에 참여하는 계산 규칙을 다룬다.

comma로 묶은 nesting parent는 `:is()`처럼 목록의 가장 높은 specificity가 nested selector에 영향을 줄 수 있다. `&`는 현재 nesting selector를 명시한다.

## 예시

```css
.card, #featured {
  & .title { color: rebeccapurple; }
}
```

`.card .title`도 selector list에 있는 `#featured`의 높은 specificity 영향을 받을 수 있으므로 ID와 class parent를 무심코 함께 묶지 않는다.

## 면접 답변 예시

> CSS nesting은 component selector와 상태 규칙을 가까이 둘 수 있지만 중첩이 specificity를 자동으로 낮춰 주는 기능은 아닙니다. Parent selector list는 `:is()`와 비슷하게 목록에서 가장 높은 specificity가 nested selector 계산에 영향을 줄 수 있어 ID와 class를 같은 목록에 섞지 않겠습니다. 중첩 깊이는 component와 state 관계가 바로 읽히는 수준으로 제한하고 낮은 우선순위가 필요하면 `:where()`나 cascade layer를 의도적으로 사용합니다. Native nesting과 build processor 변환 결과를 비교하고 DevTools에서 최종 selector, specificity와 실제 override 원인을 확인합니다.

## 장점

- BEM 선택자 반복을 줄인다.

## 단점

- 중첩 깊이가 길어지면 최종 selector를 예측하기 어렵다.

## 주의사항 / 실무 팁

- 중첩은 component·state 관계를 드러내는 정도로 제한한다.
