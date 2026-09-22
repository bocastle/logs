# CSS filter와 backdrop-filter 정리

## 핵심 요약

- 이미지·icon·배경에 비파괴적 시각 효과를 적용한다.
- 큰 영역의 blur는 paint·composite 비용과 GPU 메모리를 키운다.
- blur radius와 적용 영역을 작게 제한한다.

## 개념 설명

CSS `filter`는 요소 자신과 렌더링된 하위 콘텐츠에 그래픽 효과를 적용하고, `backdrop-filter`는 요소 뒤에 보이는 backdrop pixel에 효과를 적용한다.

filter function list의 `blur()`, `brightness()`, `drop-shadow()`는 순서대로 결과에 적용된다. `backdrop-filter`가 보이려면 전경 요소의 배경에 투명한 부분이 있어야 한다.

## 예시

```css
.photo { filter: saturate(.9) contrast(1.05); }
.toolbar {
  background: rgb(255 255 255 / .72);
  backdrop-filter: blur(12px);
}
```

photo pixel은 직접 보정하고 반투명 toolbar는 뒤쪽 페이지 pixel을 blur한다.

## 면접 답변 예시

> CSS `filter`는 element와 그 하위가 렌더링된 결과에 효과를 적용하고, `backdrop-filter`는 반투명 element 뒤에 보이는 pixel을 처리합니다. Backdrop blur는 넓은 영역이나 계속 움직이는 배경에서 paint·composite 비용과 GPU memory를 크게 쓸 수 있어 적용 면적과 radius를 제한하겠습니다. Filter가 stacking context와 containing block을 만들어 overlay 위치와 z-index에도 영향을 줄 수 있다는 점도 확인해야 합니다. 효과가 없거나 transparency를 줄이는 환경에서도 읽을 수 있도록 충분히 불투명한 background fallback과 contrast를 제공합니다.

## 장점

- glass surface의 배경과 전경 경계를 표현한다.

## 단점

- backdrop 내용이 지속적으로 바뀌면 매 프레임 처리 비용이 커진다.

## 주의사항 / 실무 팁

- 저사양 기기에서 scroll FPS와 paint time을 측정한다.
