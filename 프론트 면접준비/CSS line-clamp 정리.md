# CSS line-clamp 정리

## 핵심 요약

- 반복 card의 높이를 예측 가능하게 만든다.
- 중요한 정보와 링크 문구가 숨을 수 있다.
- 생략된 텍스트 전체를 확인할 세부 화면이나 펼치기를 제공한다.

## 개념 설명

CSS `line-clamp`는 block container의 콘텐츠를 지정한 줄 수 뒤에서 제한하고 보통 ellipsis로 나머지 텍스트가 생략됐음을 보여 주는 overflow 속성이다.

현재 호환성을 위한 `-webkit-line-clamp`는 `display: -webkit-box`와 `-webkit-box-orient: vertical`의 조합이 필요하다. 콘텐츠를 실제로 clip하려면 `overflow: hidden`도 함께 둔다.

## 예시

```css
.card-title {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  line-clamp: 3;
}
```

카드 제목을 실제 3줄로 clamp하고 초과 텍스트를 숨긴다. link 텍스트에 쓰면 문장 중간에서 잘릴 수 있다.

## 면접 답변 예시

> CSS line clamp는 card처럼 높이를 맞춰야 하는 영역에서 text를 지정한 줄 수까지만 보여 주는 방법입니다. JavaScript로 글자 수를 자르는 것보다 font, 언어와 실제 layout에 맞춰 줄을 제한할 수 있지만 중요한 단어나 link가 중간에 가려질 수 있습니다. 내용 이해에 전체 문장이 필요하면 펼치기 control이나 상세 화면을 제공하고, 화면 밖에 같은 text를 중복 배치해 보조기술이 두 번 읽게 만들지는 않겠습니다. 표준 문법과 호환용 WebKit 조합의 지원 차이를 확인하고 대표 언어와 web font 로딩 뒤 결과를 검수합니다.

## 장점

- 긴 제목이 grid 레이아웃을 밀어내는 일을 줄인다.

## 단점

- 폰트·폭·언어에 따라 보이는 텍스트 범위가 달라진다.

## 주의사항 / 실무 팁

- 대표 언어와 폰트 로딩 후의 줄 수를 검수한다.
