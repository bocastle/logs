# CSS container style query 정리

## 핵심 요약

- component variant 상태를 DOM class 전파 없이 CSS token으로 공유한다.
- custom property 오타는 query가 조용히 불일치하게 만든다.
- layout variant처럼 시각 상태에 제한한다.

## 개념 설명

CSS container style query는 query container에 적용된 custom property 값 등 style 특성을 `@container style()` 조건으로 삼아 하위 스타일을 변경하는 기능이다.

상위 component에 `--variant: compact`같은 상태 token을 놓고 `@container style(--variant: compact)`으로 하위 레이아웃을 분기한다. 일반 속성 query의 지원 범위는 따로 확인한다.

## 예시

```css
.card { container-name: card; --variant: compact; }
@container card style(--variant: compact) {
  .card-actions { display: none; }
}
```

card container의 semantic variant token이 compact일 때만 부가 action을 숨긴다.

## 면접 답변 예시

> Container style query는 ancestor container의 custom property 같은 style state를 조건으로 descendant CSS를 바꾸는 방식입니다. Component 폭에 반응하는 size query와 semantic variant token에 반응하는 style query의 책임을 나누겠습니다. Custom property typo나 미지원 browser에서는 조건이 조용히 match하지 않을 수 있어 기본 layout이 먼저 동작하고 feature query와 visual test로 enhancement를 확인해야 합니다. 업무 state와 ARIA는 DOM·JavaScript에도 명시하고 CSS token만 진실 원천으로 삼지 않습니다.

## 장점

- 하위 요소가 상위 상태에 선언적으로 반응한다.

## 단점

- semantic state를 CSS에만 두면 JS와 접근성 상태가 어긋날 수 있다.

## 주의사항 / 실무 팁

- ARIA state나 업무 상태는 DOM에도 명시한다.
