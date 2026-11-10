# CSS aspect-ratio 실무 정리

## 핵심 요약

- 미디어 로드 전 공간을 예약한다.
- width와 height를 둘 다 고정하면 aspect ratio가 크기를 바꾸지 못한다.
- `img`에는 HTML `width`/`height`도 함께 제공한다.

## 개념 설명

CSS `aspect-ratio` 실무 적용은 이미지·video·card placeholder의 선호 비율을 레이아웃 전에 알려 CLS를 줄이고 responsive 크기를 제어하는 작업이다.

적어도 한 축의 size가 auto일 때 preferred aspect ratio가 크기 계산에 참여한다. replaced element에 `auto 16 / 9`를 쓰면 intrinsic ratio가 로드된 뒤 그 비율을 사용한다.

## 예시

```css
.video {
  inline-size: 100%;
  aspect-ratio: 16 / 9;
  block-size: auto;
}
```

영상 폭은 컨테이너에 맞추고 16:9 높이를 레이아웃 시점에 계산한다.

## 면접 답변 예시

> `aspect-ratio`는 image나 video가 load되기 전에 box 비율을 알려 layout space를 예약하는 데 유용합니다. Preferred ratio는 일반적으로 적어도 한 축이 auto일 때 크기 계산에 참여하고 width와 height를 모두 고정하면 기대한 조정이 일어나지 않습니다. Replaced image에는 HTML `width`와 `height`도 제공해 초기 ratio를 parser가 일찍 알게 하겠습니다. Flex·grid의 min-content 제약과 실제 media ratio를 확인하고 placeholder, loaded와 error 상태의 CLS를 비교합니다.

## 장점

- 폭 하나로 높이를 responsive하게 계산한다.

## 단점

- 실제 콘텐츠 비율과 다르면 crop 또는 여백이 생긴다.

## 주의사항 / 실무 팁

- flex/grid item의 `min-inline-size: 0`과 overflow를 확인한다.
