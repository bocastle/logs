# CSS content 속성 정리

## 핵심 요약

- 장식 아이콘과 보조 기호를 마크업 없이 붙일 수 있다.
- 의미 있는 정보를 `content`에만 두면 accessible name에 빠질 수 있다.
- 정보 전달 텍스트는 DOM에 두고 content는 장식에 제한한다.

## 개념 설명

CSS `content` 속성은 pseudo-element나 generated content에서 장식 텍스트, 카운터, 이미지 등을 삽입하는 속성이다.

`::before`와 `::after`에 주로 쓰며, 의미 있는 문구를 CSS `content`에만 두면 accessibility tree나 번역 흐름에서 빠질 수 있다.

## 예시

```css
.required::after {
  content: " *";
  color: var(--danger);
}
```

`::before`나 `::after`의 `content`는 장식 표시에 적합하지만, 필수 입력 안내처럼 의미가 있으면 실제 텍스트와 접근성 설명도 필요하다.

## 면접 답변 예시

> CSS `content`는 `::before`와 `::after` 같은 generated content에 장식 기호, counter나 image를 넣는 속성입니다. Accessible name, 오류 메시지나 필수 여부처럼 의미 있는 정보는 노출 방식이 일관되지 않고 번역·복사에서도 빠질 수 있어 실제 DOM과 접근성 속성에 두겠습니다. Pseudo-element는 장식에만 쓰고 절대 위치한 overlay가 link나 input의 hit area를 가리지 않는지 pointer와 touch로 확인합니다. Screen reader, forced colors와 localization에서 generated content가 중복으로 읽히거나 의미를 잃지 않는지도 검수합니다.

## 장점

- CSS counters와 함께 번호 표시를 만들 수 있다.

## 단점

- 번역과 복사 동작에서 생성 텍스트가 기대와 다를 수 있다.

## 주의사항 / 실무 팁

- 스크린 리더에서 generated content 노출을 확인한다.
