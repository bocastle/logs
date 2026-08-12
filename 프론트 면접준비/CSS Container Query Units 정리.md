# CSS Container Query Units 정리

## 핵심 요약

- 재사용 컴포넌트를 viewport breakpoint에서 분리한다.
- query container가 없으면 단위 기준이 예상과 다를 수 있다.
- query container에 `container-type`을 명시한다.

## 개념 설명

CSS Container Query Units는 viewport가 아니라 선택된 query container의 크기 1%를 기준으로 하는 `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax` 단위다.

컨테이너의 inline size 1%는 `1cqi`로 표현하며 해당 축의 eligible query container를 찾아 값을 계산한다. 극단 크기를 막기 위해 `clamp()`와 함께 쓴다.

## 예시

```css
.card-list { container-type: inline-size; }
.card-title { font-size: clamp(1rem, 5cqi, 1.5rem); }
```

같은 카드 제목이 sidebar와 main grid에서 자신의 query container 폭에 맞게 크기를 조정한다.

## 면접 답변 예시

> Container Query Unit은 viewport가 아니라 선택된 query container 크기의 1%를 기준으로 합니다. 같은 card가 sidebar와 main grid에 놓여도 자신의 container 폭에 맞춰 spacing과 type을 조절할 수 있습니다. Font size를 container에 무제한으로 연결하면 가독성을 해칠 수 있어 `clamp()`로 최소·최대를 둡니다. Nested container에서는 어떤 요소가 기준으로 선택되는지와 대표 폭을 모두 테스트하겠습니다.

## 장점

- 지역 레이아웃 크기에 맞는 spacing과 type scale을 만든다.

## 단점

- nested container에서 선택된 기준 요소를 혼동하기 쉽다.

## 주의사항 / 실무 팁

- `clamp()`로 최소·최대 크기를 제한한다.
