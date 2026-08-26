# CSS object-fit과 aspect-ratio 정리

## 핵심 요약

- 이미지 로드 전 레이아웃 공간을 확보한다.
- `cover`는 인물이나 텍스트가 잘릴 수 있다.
- 콘텐츠 중요도에 따라 cover와 contain을 고른다.

## 개념 설명

CSS `aspect-ratio`는 요소 박스의 선호 가로세로 비율을, `object-fit`은 교체 콘텐츠가 그 박스 안을 채우는 방식을 정한다.

`aspect-ratio` 또는 HTML `width`/`height`로 박스 크기를 먼저 확보하고 `object-fit: cover`로 crop, `contain`으로 전체 표시를 선택한다. `object-position`이 crop 초점을 정한다.

## 예시

```css
.thumbnail {
  inline-size: 100%;
  aspect-ratio: 4 / 3;
  object-fit: cover;
  object-position: center;
}
```

4:3 썸네일 박스를 먼저 잡고 이미지를 포스터 영역에 맞게 crop한다.

## 면접 답변 예시

> `aspect-ratio`는 image가 로드되기 전부터 box 비율을 확보하고, `object-fit`은 그 안에 교체 콘텐츠를 어떻게 맞출지 정합니다. Thumbnail처럼 frame을 꽉 채워야 하면 `cover`, 원본 전체가 중요하면 `contain`을 선택하겠습니다. `cover`는 찌그러뜨리지는 않지만 사람 얼굴이나 글자를 crop할 수 있어 `object-position`으로 초점을 조절해야 합니다. Background image에는 `object-fit`이 적용되지 않으며, 세로·가로 원본과 responsive 폭을 실제 화면에서 확인합니다.

## 장점

- 다른 원본 비율을 일관된 card frame에 배치한다.

## 단점

- 잘못된 aspect ratio는 원본을 찌그러뜨리지는 않지만 과도하게 crop한다.

## 주의사항 / 실무 팁

- 인물 이미지는 `object-position`을 데이터로 조정할 수 있게 한다.
