# CSS color-scheme 정리

## 핵심 요약

- native 폼 컨트롤을 테마와 맞춘다.
- `color-scheme`만으로 커스텀 색 token이 바뀌지는 않는다.
- DevTools에서 native input과 scrollbar까지 확인한다.

## 개념 설명

CSS `color-scheme`은 요소가 light·dark 색 계획을 지원한다고 브라우저에 알려 native form control, scrollbar, canvas color의 기본 표현을 맞추는 속성이다.

`:root { color-scheme: light dark; }`는 두 scheme을 허용하고 `.dark-panel { color-scheme: dark; }`는 해당 subtree의 UA 색을 dark로 제한한다. 애플리케이션 token은 별도로 제공해야 한다.

## 예시

```css
:root { color-scheme: light dark; }
[data-theme="dark"] {
  color-scheme: dark;
  --surface: #16181d;
}
```

dark theme subtree에 native control 색 계획과 제품 surface token을 함께 설정한다.

## 면접 답변 예시

> `color-scheme`은 해당 element나 subtree가 light와 dark 중 어떤 색 체계를 지원하는지 browser에 알려 native input, scrollbar와 기본 canvas 색을 맞추는 속성입니다. 제품의 custom color token을 자동으로 바꾸는 기능은 아니므로 실제 theme state에서 token과 `color-scheme`을 함께 갱신하겠습니다. Chart, image와 직접 그린 canvas도 별도 theme 처리가 필요합니다. Native control까지 포함해 light·dark와 forced colors 환경을 확인하고 사용자 선택과 선언 순서가 어긋나지 않게 합니다.

## 장점

- 스크롤바와 기본 canvas 색의 불일치를 줄인다.

## 단점

- scheme 순서와 사용자 테마 선택이 어긋날 수 있다.

## 주의사항 / 실무 팁

- 테마 token과 `color-scheme`을 같은 root 상태에 연결한다.
