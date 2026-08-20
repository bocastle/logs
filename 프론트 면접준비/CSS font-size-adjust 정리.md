# CSS font-size-adjust 정리

## 핵심 요약

- fallback 기간의 텍스트 가독성을 높인다.
- metric 값이 잘못되면 fallback이 과도하게 커지거나 작아진다.
- primary·fallback font 조합별로 줄바꿈을 비교한다.

## 개념 설명

CSS `font-size-adjust`는 primary font와 fallback font의 x-height 등 지정한 font metric 비율을 맞춰 같은 `font-size`에서 소문자의 체감 크기를 유지하는 속성이다.

기본 metric은 ex-height이며 number 또는 `from-font`로 aspect value를 정한다. 브라우저는 선택된 font의 metric에 맞춰 사용 font size를 조정한다.

## 예시

```css
body {
  font-family: "Brand Sans", Arial, sans-serif;
  font-size-adjust: from-font;
}
```

Brand Sans가 로드되기 전 fallback의 소문자 체감 크기를 primary font metric에 가깝게 맞춘다.

## 면접 답변 예시

> `font-size-adjust`는 같은 `font-size`라도 font마다 다른 x-height 같은 metric을 보정해 fallback과 web font의 체감 크기를 가깝게 만드는 속성입니다. Font가 늦게 로드되는 동안에도 소문자 가독성을 유지하고 줄바꿈 차이를 줄이는 데 도움을 줄 수 있습니다. 다만 글자 폭과 line metric 전체를 같게 만드는 기능은 아니라서 이것만으로 CLS가 사라진다고 보지는 않습니다. 실제 primary·fallback 조합에서 줄바꿈과 CLS를 측정하고 `from-font` 지원 여부도 확인하겠습니다.

## 장점

- font swap 전후 줄바꿈 차이를 줄일 수 있다.

## 단점

- x-height만 맞춰도 글자 폭과 줄 높이는 다를 수 있다.

## 주의사항 / 실무 팁

- `from-font` 지원과 fallback number를 확인한다.
