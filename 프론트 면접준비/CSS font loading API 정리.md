# CSS font loading API 정리

## 핵심 요약

- canvas 측정과 스크린샷 시점을 폰트 준비 후로 맞춘다.
- `document.fonts.ready`를 전체 UI 표시의 필수 관문으로 쓰면 느려진다.
- 일반 텍스트는 CSS `font-display`를 기본으로 쓴다.

## 개념 설명

CSS Font Loading API는 `FontFace`와 `document.fonts` `FontFaceSet`으로 폰트 로딩·상태·완료를 JavaScript에서 관찰하거나 제어하는 API다.

`document.fonts.load()`는 특정 font shorthand와 텍스트에 필요한 face를 로드하고 `document.fonts.ready`는 현재 font load와 레이아웃이 안정될 때 resolve된다.

## 예시

```js
await document.fonts.load('600 1rem "Article Sans"', '제목');
if (document.fonts.check('600 1rem "Article Sans"')) revealHeadline();
```

제목에 필요한 weight와 glyph를 먼저 로드하고 사용 가능 상태를 확인한다.

## 면접 답변 예시

> CSS Font Loading API는 특정 face의 준비 시점을 JavaScript에서 확인해야 하는 canvas, editor와 visual regression에 적합합니다. `document.fonts.load()`가 resolve돼도 fallback과 같은 metric이나 줄바꿈을 보장하는 것은 아니므로 실제 render 결과를 별도로 확인하겠습니다. 일반 text 전체를 `document.fonts.ready` 뒤에 숨기면 content 표시가 느려져 CSS `font-display`를 기본으로 사용합니다. 필요한 weight와 glyph만 load하고 `loadingerror`, timeout과 fallback 상태를 test합니다.

## 장점

- 필요한 font face만 명시적으로 로드한다.

## 단점

- 사용자가 보지 않는 face까지 미리 로드하면 byte를 낭비한다.

## 주의사항 / 실무 팁

- API는 canvas·editor·visual regression처럼 시점 제어가 필요한 곳에 제한한다.
