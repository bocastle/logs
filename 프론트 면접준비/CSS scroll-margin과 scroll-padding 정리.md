# CSS scroll-margin과 scroll-padding 정리

## 핵심 요약

- anchor target이 fixed header 뒤에 가리는 문제를 줄인다.
- target margin과 container padding을 중복 적용하면 여백이 과해진다.
- header 높이를 custom property로 공유한다.

## 개념 설명

CSS `scroll-margin`은 snap·anchor target 주변의 외부 offset을, `scroll-padding`은 scroll container의 최적 보이는 영역 안쪽 offset을 정하는 속성이다.

fixed header 높이는 scroller에 `scroll-padding-block-start`, 개별 heading 여백은 target에 `scroll-margin-block-start`로 표현한다. hash navigation·`scrollIntoView()`·scroll snap이 이 offset을 참조한다.

## 예시

```css
html { scroll-padding-block-start: 4.5rem; }
article h2 { scroll-margin-block-start: 1rem; }
```

hash로 h2에 이동할 때 fixed header 아래 4.5rem 영역과 heading 자체 1rem 여백을 고려한다.

## 면접 답변 예시

> `scroll-padding`은 scroll container 안의 최적 visible 영역을, `scroll-margin`은 개별 target 주변의 scroll offset을 정합니다. Fixed header 높이는 container의 `scroll-padding-block-start`에 두고 특정 heading의 추가 간격만 target margin으로 나눠 중복 offset을 피하겠습니다. JavaScript에서 header 높이를 빼는 계산보다 hash navigation과 `scrollIntoView()`, focus 이동에 같은 정책을 적용하기 쉽습니다. Nested scroller와 responsive header token을 실제 크기와 맞추고 각 이동 경로를 따로 테스트합니다.

## 장점

- JS에서 header 높이를 빼는 scroll 계산을 줄인다.

## 단점

- nested scroller에서 어느 container의 padding이 적용되는지 혼동할 수 있다.

## 주의사항 / 실무 팁

- hash, focus, `scrollIntoView()`, snap 이동을 각각 테스트한다.
