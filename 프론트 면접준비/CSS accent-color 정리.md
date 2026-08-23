# CSS accent-color 정리

## 핵심 요약

- native 컨트롤의 테마 일관성을 높인다.
- 브랜드 color의 대비가 부족하면 선택 상태를 읽기 어렵다.
- checkbox·radio·range·progress를 light·dark에서 검수한다.

## 개념 설명

CSS `accent-color`는 checkbox, radio, range, progress 등 일부 native form control의 emphasis color를 테마에 맞게 지정하는 속성이다.

색 하나를 제공하면 브라우저가 컨트롤의 나머지 표현을 유지하며 필요할 때 가독성을 위해 조정할 수 있다.

## 예시

```css
:root { accent-color: var(--accent); }
@media (forced-colors: active) {
  :root { accent-color: auto; }
}
```

native control의 accent를 테마 token에 연결하고 forced colors에서는 플랫폼 판단을 유지한다.

## 면접 답변 예시

> `accent-color`는 checkbox나 radio 같은 native control의 강조색을 theme에 맞추면서도 browser의 기본 상호작용을 유지하는 방법입니다. 직접 control을 다시 그리는 것보다 focus, disabled 상태와 platform 동작을 보존하기 쉽습니다. 다만 색 하나를 지정할 뿐 label, error message나 keyboard focus를 대신하지는 않습니다. Light·dark theme에서 선택 상태의 대비를 확인하고 forced colors에서는 사용자와 UA의 color 결정을 방해하지 않겠습니다.

## 장점

- 커스텀 checkbox 구현을 줄일 수 있다.

## 단점

- 컨트롤 종류와 브라우저별 표현 차이가 남는다.

## 주의사항 / 실무 팁

- focus ring을 별도로 유지한다.
