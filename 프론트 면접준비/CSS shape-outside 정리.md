# CSS shape-outside 정리

## 핵심 요약

- 잡지형 레이아웃이나 소개 페이지의 텍스트 흐름을 풍부하게 만들 수 있다.
- float 기반이라 일반 grid/flex 레이아웃과 책임이 충돌할 수 있다.
- 본문 가독성이 유지되는 폭에서만 적용한다.

## 개념 설명

`shape-outside`는 float된 요소 주변의 inline 콘텐츠가 사각형이 아닌 원, polygon, 이미지 alpha 같은 형태를 따라 감기게 하는 CSS 속성이다.

텍스트 wrapping은 float 요소에 적용된 shape를 기준으로 계산된다. 현대 앱 UI보다 editorial 레이아웃이나 프로필 이미지 주변 본문 흐름에 주로 맞다.

## 예시

```css
.avatar {
  float: inline-start;
  shape-outside: circle(50%);
  clip-path: circle(50%);
}
```

`shape-outside`와 `float`로 원형 이미지 주변에 본문 wrapping을 만드는 예다.

## 면접 답변 예시

> `shape-outside`는 float element 주변의 inline text가 사각형 대신 circle이나 polygon 같은 형태를 따라 흐르게 합니다. Image를 실제로 자르는 `clip-path`와 text wrapping shape는 별도이므로 두 값이 어긋나지 않게 같은 token에서 관리하겠습니다. 일반 application layout보다 editorial content에 적합하고 좁은 화면에서는 한 줄이 지나치게 짧아질 수 있습니다. Mobile에서는 shape와 float를 해제하는 fallback을 두고 여러 content 길이와 language에서 가독성을 확인합니다.

## 장점

- 이미지 실루엣에 맞춘 wrapping을 CSS로 표현한다.

## 단점

- 좁은 화면에서는 줄 길이가 지나치게 짧아질 수 있다.

## 주의사항 / 실무 팁

- 모바일에서는 shape를 끄는 fallback을 둔다.
