# CSS Dynamic Viewport Units 정리

## 핵심 요약

- `sv*`, `lv*`, `dv*`는 mobile browser UI의 작은·큰·현재 viewport를 구분하는 CSS 단위다.
- 안정적인 page shell은 `svh`, 현재 화면을 채울 overlay는 `dvh`처럼 layout 목적에 맞춰 선택한다.
- Dynamic 값은 scroll 중 바뀔 수 있고 safe area·virtual keyboard 문제를 모두 해결하지는 않는다.

## 개념 설명

Mobile browser의 주소창과 toolbar는 scroll에 따라 열리고 닫혀 viewport 높이를 바꾼다. 새로운 viewport 단위는 browser UI가 열린 작은 크기, 닫힌 큰 크기와 현재 동적 크기를 따로 표현한다.

`100svh`는 가장 작은 높이, `100lvh`는 가장 큰 높이, `100dvh`는 현재 값을 따른다. `dvh`를 전체 page에 사용하면 toolbar가 움직일 때 layout도 계속 변할 수 있어 fullscreen overlay처럼 현재 화면과 맞아야 하는 영역에 제한하는 편이 좋다.

## 예시

```css
.hero { min-block-size: 100svh; }
.overlay { block-size: 100dvh; }
```

Hero는 browser UI가 보여도 들어오는 안정적인 최소 높이를 사용하고 overlay는 현재 보이는 영역을 따른다. 하단 CTA에는 `env(safe-area-inset-bottom)`과 keyboard focus 처리를 별도로 검토한다.

## 면접 답변 예시

> Dynamic viewport unit은 mobile browser UI 상태에 따른 작은·큰·현재 viewport를 구분합니다. 높이가 안정적이어야 하는 page shell에는 `svh`, 현재 화면을 채우는 overlay에는 `dvh`를 사용하겠습니다. `dvh`는 scroll 중 값이 바뀔 수 있고 safe area와 keyboard를 자동으로 해결하지는 않습니다. `vh` fallback을 두고 실제 iOS·Android에서 toolbar, 회전과 keyboard 상태를 확인합니다.

## 장점

- Component별로 안정적인 크기와 현재 보이는 크기 중 필요한 기준을 선택할 수 있다.
- JavaScript resize listener 없이 browser UI 변화를 CSS에 반영할 수 있다.

## 단점

- `dvh`는 scroll 중 값이 변해 큰 영역에서 layout 흔들림을 만들 수 있다.
- 오래된 browser에는 `vh` fallback이 필요하고 virtual keyboard 정책은 별도로 처리해야 한다.

## 주의사항 / 실무 팁

- `min-height: 100vh` fallback 뒤에 지원되는 새 단위를 덧쓴다.
- Toolbar 열림·닫힘, 회전, keyboard와 safe area가 있는 실제 device에서 검수한다.
