# CSS subpixel rendering 정리

## 핵심 요약

- 소수 단위로 부드러운 배치와 애니메이션을 만든다.
- fractional translate가 텍스트에 남으면 glyph가 흐려 보일 수 있다.
- 텍스트 parent에 불필요한 fractional transform이 남지 않게 한다.

## 개념 설명

CSS subpixel rendering은 레이아웃과 transform이 생성한 소수 CSS pixel 좌표가 device pixel grid에 매핑될 때 경계와 텍스트가 anti-aliasing되는 현상을 다룬다.

percentage·fractional `fr`·scale·translate가 0.5px 같은 좌표를 만들면 브라우저는 device pixel ratio에 맞게 coverage를 분배한다. 일반 layout을 무조건 integer로 반올림하지 않는다.

## 예시

```css
.hairline {
  block-size: 1px;
  transform: scaleY(.5);
  transform-origin: top;
}
```

DPR이 높은 화면에서 더 얇은 경계를 만들 수 있지만 DPR·zoom별 선명도가 다를 수 있다.

## 면접 답변 예시

> CSS layout과 transform은 소수 CSS pixel 위치를 만들 수 있고 browser는 이를 device pixel coverage로 나눠 그립니다. 그래서 fractional grid와 translate는 부드러운 배치에 유용하지만 text parent에 남은 소수 transform은 glyph가 흐려 보일 수 있습니다. 모든 값을 강제로 정수 반올림하기보다 실제 gap이나 overflow인지 screenshot anti-aliasing 차이인지 먼저 구분하겠습니다. 1x·2x DPR과 여러 zoom에서 경계와 text를 확인하고 성능 근거 없이 transform을 추가하지 않습니다.

## 장점

- 여러 DPR에서 시각적 정밀도를 높인다.

## 단점

- 자식 폭의 반올림 오차가 반복 grid에서 1px 틈을 만들 수 있다.

## 주의사항 / 실무 팁

- 1x·2x DPR과 90%·125% zoom에서 경계와 텍스트를 검수한다.
