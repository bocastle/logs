# CSS hyphens와 word-break 정리

## 핵심 요약

- 긴 단어가 좁은 column을 깨뜨리는 일을 줄인다.
- `lang`이 없으면 auto hyphenation이 잘못되거나 작동하지 않는다.
- document와 문단에 올바른 `lang`을 지정한다.

## 개념 설명

CSS `hyphens`는 언어별 하이픈 위치에서 단어를 나눌지 정하고, `word-break`는 CJK 및 긴 토큰의 일반 단어 바꿈 규칙을 조정한다.

`hyphens: auto`는 올바른 `lang`을 기준으로 사전의 하이프네이션 기회를 사용한다. `word-break: break-all`은 non-CJK 문자 사이도 나눌 수 있어 긴 URL에는 `overflow-wrap: anywhere`가 더 자연스러울 수 있다.

## 예시

```css
.article { hyphens: auto; }
.product-code { overflow-wrap: anywhere; }
:lang(ko) .title { word-break: keep-all; }
```

본문은 언어 사전을 사용하고 product code는 overflow를 막으며 한국어 제목은 어절 중심 줄바꿈을 선택한다.

## 면접 답변 예시

> `hyphens`는 언어 사전에 맞는 단어 분할을, `word-break`와 `overflow-wrap`은 일반 줄바꿈과 긴 token의 overflow 정책을 다룹니다. 본문에는 정확한 `lang`과 `hyphens: auto`를 사용하고 URL이나 product code에는 `overflow-wrap: anywhere`를 제한적으로 적용하겠습니다. `break-all`은 일반 단어까지 임의로 나눠 가독성을 해칠 수 있고 한국어 `keep-all`도 긴 어절에서는 overflow가 날 수 있습니다. 한국어, 영어와 독일어 본문 및 URL sample을 함께 검수합니다.

## 장점

- 언어별 읽기 규칙을 CSS에 반영한다.

## 단점

- `break-all`은 단어를 무작위로 쪼개 가독성을 해친다.

## 주의사항 / 실무 팁

- hyphenation은 본문에, anywhere는 긴 기술 token에 제한한다.
