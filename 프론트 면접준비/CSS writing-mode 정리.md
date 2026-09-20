# CSS writing-mode 정리

## 핵심 요약

- 세로쓰기 언어와 편집 디자인을 native text flow로 표현한다.
- left·right·width·height에 묶인 스타일은 세로쓰기에서 어긋난다.
- physical property보다 logical property를 우선한다.

## 개념 설명

CSS `writing-mode`은 block flow direction과 inline 텍스트 진행 방향을 horizontal-tb, vertical-rl, vertical-lr 등으로 정하는 속성이다.

writing mode가 바뀌면 block axis와 inline axis가 달라지며 logical size·margin·padding·inset이 그 축을 따른다. glyph 방향은 `text-orientation`과 함께 제어한다.

## 예시

```css
.vertical-label {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  margin-inline-end: .5rem;
}
```

세로쓰기 label의 텍스트 방향을 vertical-rl로 두고 여백을 inline axis 기준으로 둔다.

## 면접 답변 예시

> `writing-mode`은 단순히 글자를 회전하는 것이 아니라 block과 inline flow 자체를 가로쓰기나 세로쓰기로 바꾸는 CSS 속성입니다. 세로쓰기에서는 left, width 같은 physical property보다 `margin-inline`, `block-size` 같은 logical property를 써야 같은 component를 재사용하기 쉽습니다. Latin 약어와 숫자의 glyph 방향은 `text-orientation`과 실제 언어 조합을 보고 결정하겠습니다. Scroll, text selection, focus ring과 overflow 방향도 달라질 수 있어 한국어·일본어·Latin 혼합 content로 검증합니다.

## 장점

- logical property와 함께 다국어 레이아웃을 재사용한다.

## 단점

- Latin acronym과 number의 glyph 방향이 언어별로 어색할 수 있다.

## 주의사항 / 실무 팁

- 한국어·일본어·Latin·숫자 조합을 검수한다.
