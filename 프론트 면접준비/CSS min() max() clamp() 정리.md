# CSS min() max() clamp() 정리

## 핵심 요약

- media query 없이도 유동 값의 극단을 막을 수 있다.
- 계산식이 길어지면 실제 값 범위를 읽기 어렵다.
- 최소와 최대값을 먼저 정하고 선호 값을 넣는다.

## 개념 설명

`min()`, `max()`, `clamp()`는 CSS 값의 하한과 상한을 선언 안에서 직접 계산하는 comparison function이다.

`min()`은 후보 중 작은 값을, `max()`는 큰 값을 고르고, `clamp()`는 최소·선호·최대 범위 안에 값을 묶어 반응형 spacing과 폭에 자주 쓴다.

## 예시

```css
.card {
  inline-size: min(100%, 42rem);
  padding-inline: clamp(1rem, 4vw, 2.5rem);
  margin-block: max(1rem, 2vh);
}
```

`min()`, `max()`, `clamp()`로 카드 폭과 여백을 디자인 경계 안에 두는 예다.

## 면접 답변 예시

> CSS `min()`, `max()`, `clamp()`는 media query를 여러 번 나누지 않고도 값의 경계를 선언하는 함수입니다. `clamp(min, preferred, max)`로 spacing이나 폭이 viewport에 따라 변하되 design 최소·최대를 넘지 않게 할 수 있습니다. 먼저 접근성과 layout에 필요한 양 끝 값을 정하고 그 사이의 preferred 값을 넣겠습니다. 계산이 길어지면 custom property로 의도를 이름 붙이고 대표 viewport의 computed value를 확인합니다.

## 장점

- 디자인 최소·최대 제약을 CSS 선언에 직접 남긴다.

## 단점

- 글자 크기까지 과하게 유동화하면 접근성과 검수가 어려워진다.

## 주의사항 / 실무 팁

- 대표 viewport에서 computed value를 확인한다.
