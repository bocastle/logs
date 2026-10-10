# CSS font-display 정리

## 핵심 요약

- 웹폰트 로딩 중 빈 텍스트를 줄인다.
- swap은 font metric 차이로 layout shift를 만들 수 있다.
- 본문·아이콘·브랜드 display font의 정책을 따로 정한다.

## 개념 설명

CSS `font-display`는 `@font-face`의 웹폰트가 로드되는 동안 block, swap, fallback 기간을 어떻게 운영할지 정하는 descriptor다.

`block`, `swap`, `fallback`, `optional`은 텍스트를 숨기는 기간과 fallback에서 webfont로 교체할 기간을 다르게 정한다. 본문은 보통 즉시 표시를 우선한다.

## 예시

```css
@font-face {
  font-family: "Article Sans";
  src: url("/fonts/article.woff2") format("woff2");
  font-display: swap;
}
```

fallback로 본문을 즉시 보여 준 뒤 Article Sans가 준비되면 swap한다.

## 면접 답변 예시

> `font-display`는 web font가 준비되기 전 text를 숨길지 fallback으로 먼저 보여 줄지 정하는 `@font-face` descriptor입니다. 본문은 보통 읽을 수 있는 상태를 우선해 swap이나 fallback을 검토하지만 brand display font와 icon font는 같은 정책이 맞지 않을 수 있습니다. Swap은 FOIT를 줄이는 대신 fallback과 metric 차이로 CLS를 만들 수 있어 metric override와 `font-size-adjust`를 함께 보겠습니다. Cold cache와 느린 network에서 FOIT, FOUT, CLS와 실제 font 사용 여부를 측정해 content별 정책을 정합니다.

## 장점

- 콘텐츠 중요도에 맞게 표시 정책을 고른다.

## 단점

- optional은 현재 navigation에서 webfont를 쓰지 않을 수 있다.

## 주의사항 / 실무 팁

- fallback metric override와 `font-size-adjust`를 함께 검토한다.
