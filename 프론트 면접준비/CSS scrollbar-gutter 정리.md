# CSS scrollbar-gutter 정리

## 핵심 요약

- scrollbar 등장·제거 시 폭 변화를 줄인다.
- overlay scrollbar 환경에서는 예약 효과가 보이지 않을 수 있다.
- scrollbar가 생겼다 사라지는 상태를 회귀 캡처한다.

## 개념 설명

CSS `scrollbar-gutter`는 classic scrollbar가 들어올 가능성이 있는 요소의 내부 가장자리에 gutter 공간을 미리 예약하는 속성이다.

`stable`은 overflow가 없어도 classic scrollbar gutter를 유지하고 `both-edges`를 더하면 inline 축 양쪽에 대칭 공간을 만든다. overlay scrollbar는 레이아웃 공간을 차지하지 않는다.

## 예시

```css
.modal-body {
  overflow: auto;
  scrollbar-gutter: stable both-edges;
}
```

모달 내용이 길어져 classic scrollbar가 나타나도 양쪽 여백과 중앙 정렬이 크게 흔들리지 않게 한다.

## 면접 답변 예시

> `scrollbar-gutter`는 classic scrollbar가 생길 가능성이 있는 영역에 공간을 미리 예약해 content 폭이 갑자기 바뀌는 것을 줄입니다. `stable`은 overflow가 없을 때도 gutter를 유지하고, 중앙 정렬이 중요한 scroller에는 `both-edges`로 양쪽 균형을 맞출 수 있습니다. 다만 overlay scrollbar 환경에서는 같은 예약 효과가 보이지 않고 작은 container에서는 실제 content 폭이 줄어듭니다. Body와 nested scroller에 무조건 중복 적용하지 않고 Windows classic과 macOS overlay 환경에서 전후 상태를 확인하겠습니다.

## 장점

- 중앙 정렬 UI의 시각적 안정성을 높인다.

## 단점

- `both-edges`는 작은 컨테이너의 실제 콘텐츠 폭을 줄인다.

## 주의사항 / 실무 팁

- Windows classic scrollbar와 macOS overlay scrollbar를 모두 확인한다.
