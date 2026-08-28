# CSS focus-visible 정리

## 핵심 요약

- 키보드 사용자의 현재 위치를 표시한다.
- `outline: none`만 두면 키보드 위치가 사라진다.
- Tab·Shift+Tab·focus 복귀 경로를 모두 확인한다.

## 개념 설명

CSS `:focus-visible`은 요소가 focus되었고 브라우저 heuristic상 현재 keyboard 등 입력 맥락에 focus indicator가 필요할 때 매칭되는 pseudo-class다.

키보드 탐색과 텍스트 입력 컨트롤에서 주로 매칭되며 pointer click 후의 매칭은 요소와 브라우저 판단에 따른다. `:focus`를 전역으로 제거하지 않는다.

## 예시

```css
.button:focus-visible {
  outline: 3px solid CanvasText;
  outline-offset: 3px;
}
```

키보드 focus가 보일 필요가 있을 때 버튼 경계 밖에 명확한 focus ring을 그린다.

## 면접 답변 예시

> `:focus-visible`은 element가 focus된 상태 중 browser가 keyboard 같은 입력 맥락에 focus indicator가 필요하다고 판단할 때 적용됩니다. 이를 사용하면 pointer click마다 ring을 강제하지 않으면서 keyboard 사용자의 현재 위치는 분명히 보여 줄 수 있습니다. 그렇다고 전역 `:focus` outline을 먼저 없애고 지원을 기대해서는 안 되며, text input과 programmatic focus 이동도 실제 browser에서 확인하겠습니다. Ring은 forced colors에서도 보이는 outline과 system color를 우선하고 disabled나 loading 상태에 가려지지 않게 합니다.

## 장점

- pointer 사용 시 불필요한 ring을 줄일 수 있다.

## 단점

- box-shadow ring은 forced colors에서 사라질 수 있다.

## 주의사항 / 실무 팁

- outline color에 system color fallback을 둔다.
