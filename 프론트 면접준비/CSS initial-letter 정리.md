# CSS initial-letter 정리

## 핵심 요약

- float 트릭보다 폰트 baseline에 맞는 drop cap을 만든다.
- 폰트와 언어에 따라 겹침·여백 결과가 크게 다를 수 있다.
- 긴 본문의 첫 문단처럼 명확한 편집 맥락에만 쓴다.

## 개념 설명

CSS `initial-letter`는 문단의 첫 글자가 몇 줄 높이를 차지하고 어느 baseline에 앉을지 지정해 dropped·raised cap을 만드는 속성이다.

`initial-letter: 3 2`의 첫 값은 글자 크기가 차지할 줄 수, 두 번째 값은 앉힐 baseline 위치를 나타낸다. 보통 `::first-letter`에 적용한다.

## 예시

```css
.article > p:first-of-type::first-letter {
  initial-letter: 3 2;
  color: var(--accent);
  margin-inline-end: .25rem;
}
```

첫 문단의 첫 글자를 3줄 크기로 키우고 두 번째 baseline까지 내려 에디토리얼 drop cap을 만든다.

## 면접 답변 예시

> `initial-letter`는 문단 첫 글자의 높이와 baseline 위치를 줄 수로 지정해 markup을 바꾸지 않고 drop cap을 만드는 속성입니다. `::first-letter`에 적용하되 본문 의미나 읽기 순서는 그대로 두고 지원하지 않는 환경에서는 평범한 첫 글자로 남는 progressive enhancement로 사용하겠습니다. Font metric과 언어에 따라 줄 겹침이 달라지고 첫 문자가 따옴표·숫자·emoji면 장식 의도가 깨질 수 있습니다. `@supports`로 효과를 분리하고 한국어·Latin·숫자·따옴표, 좁은 화면과 세로 쓰기에서 실제 font로 검수합니다.

## 장점

- writing mode과 font metric에 따라 첫 글자를 배치한다.

## 단점

- 브라우저 지원이 없는 환경에서 기본 `::first-letter` 스타일만 남을 수 있다.

## 주의사항 / 실무 팁

- `@supports (initial-letter: 2)`로 enhancement를 분리한다.
