# CSS place-content와 alignment 정리

## 핵심 요약

- 두 축의 content distribution을 한 선에 명시한다.
- container에 남는 공간이 없으면 distribution 효과가 보이지 않는다.
- container에 정의된 여유 공간이 있는지 먼저 확인한다.

## 개념 설명

CSS `place-content`는 grid, multi-line flex 등 alignment container 안의 전체 track·line 묶음을 block axis와 inline axis에 배치하는 `align-content`/`justify-content` shorthand다.

첫 번째 값은 `align-content`, 두 번째 값은 `justify-content`에 해당하며 하나만 쓰면 문법에 따라 두 축에 복제된다. item 자체 정렬인 `place-items`와 구분한다.

## 예시

```css
.dashboard {
  display: grid;
  min-block-size: 30rem;
  place-content: center space-between;
}
```

grid track 묶음은 block axis 중앙에 두고 inline axis에서는 남은 공간을 track 사이에 나눈다.

## 면접 답변 예시

> `place-content`는 grid track이나 여러 flex line의 묶음을 container의 block·inline 두 축에 배치하는 shorthand입니다. 개별 item을 자기 area 안에서 정렬하는 `place-items`와 역할이 다릅니다. 남는 공간이 없거나 single-line flex의 cross axis처럼 content distribution이 적용되지 않는 조건에서는 값을 바꿔도 효과가 보이지 않을 수 있습니다. Writing mode의 logical axis와 실제 free space를 먼저 확인하고 DevTools overlay로 track, line과 item 정렬을 나눠 보겠습니다.

## 장점

- logical axis 기준이라 writing mode에 적응한다.

## 단점

- single-line flex에서 `align-content`는 기대한 효과가 없다.

## 주의사항 / 실무 팁

- block·inline axis를 writing mode 기준으로 설명한다.
