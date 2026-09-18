# CSS isolation 속성 정리

## 핵심 요약

- blend 효과가 바깥 page 배경으로 새는 것을 막는다.
- isolate된 descendant의 큰 z-index도 바깥 sibling 위로 넘지 못할 수 있다.
- blend·z-index 격리 의도가 있는 component root에만 쓴다.

## 개념 설명

CSS `isolation`은 요소가 새 stacking context를 만들어 descendant blending과 z-index 비교 범위를 지역 경계안으로 격리할지 정하는 속성이다.

`isolation: isolate`는 외부 배경을 blending backdrop에서 끊고 자신을 하나의 stacking context root로 만든다. `auto`는 다른 속성이 필요할 때만 stacking context를 만든다.

## 예시

```css
.card { isolation: isolate; }
.card__art { mix-blend-mode: multiply; }
.card__badge { position: absolute; z-index: 1; }
```

card 내부 artwork의 blend와 badge z-index를 card stacking context 안에 격리한다.

## 면접 답변 예시

> transform을 트릭으로 쓰지 않고 stacking context 의도를 직접 표현한다. isolation은 clipping이나 paint containment를 제공하지 않는다. clipping이 필요하면 `overflow`/`clip-path`, paint 격리는 `contain`을 별도로 고른다.

## 장점

- component 내부 z-index 숫자를 지역적으로 관리한다.

## 단점

- 불필요한 stacking context가 많아지면 overlay 디버깅이 복잡해진다.

## 주의사항 / 실무 팁

- DevTools stacking context 계층에서 외부 overlay와의 관계를 확인한다.
