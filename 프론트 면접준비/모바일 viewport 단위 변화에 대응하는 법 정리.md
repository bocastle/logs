# 모바일 viewport 단위 변화에 대응하는 법 정리

## 핵심 요약

- `svh`, `lvh`, `dvh`는 모바일 browser UI가 열리고 닫힐 때 서로 다른 viewport 높이를 표현한다.
- 안정적인 shell에는 `svh`, 현재 보이는 영역을 따라야 하는 overlay에는 `dvh`처럼 목적에 맞춰 선택한다.
- Safe area와 virtual keyboard는 viewport 단위만으로 해결되지 않아 실제 기기의 Visual Viewport 변화까지 확인한다.

## 개념 설명

모바일 브라우저는 스크롤에 따라 주소창과 toolbar가 나타나고 사라져 보이는 높이가 바뀐다. 기존 `100vh`는 브라우저마다 어느 상태의 높이를 가리키는지 달라 fullscreen panel과 하단 CTA가 가려질 수 있다.

`svh`는 browser UI가 보일 때의 작은 viewport, `lvh`는 숨었을 때의 큰 viewport, `dvh`는 현재 동적 값을 따른다. Scroll 중 계속 크기가 바뀌면 레이아웃이 흔들릴 수 있으므로 모든 container에 `dvh`를 쓰지 않는다. 하단 UI는 safe-area inset을 한 번 적용하고 keyboard 동작은 VisualViewport API의 `window.visualViewport`와 실제 browser 정책을 확인한다.

## 예시

```css
.sheet {
  min-block-size: 100svh;
  max-block-size: 100dvh;
  padding-block-end: env(safe-area-inset-bottom);
}
```

하단 sheet의 최소 공간은 작은 viewport를 기준으로 확보하고 현재 보이는 높이를 넘지 않게 한 예다. Content가 긴 경우 overflow와 keyboard focus scroll도 별도로 처리해야 한다.

## 면접 답변 예시

> 모바일 viewport는 주소창과 toolbar 상태에 따라 높이가 달라집니다. 안정적인 page shell에는 `svh`, 현재 보이는 영역을 따라야 하는 overlay에는 `dvh`를 사용하는 식으로 목적을 나누겠습니다. 하단 고정 UI에는 safe-area inset을 한 번만 적용하고, virtual keyboard는 viewport unit만 믿지 않고 focus와 `window.visualViewport` 변화를 확인합니다. 주소창 열림·닫힘, 키보드, 회전 상태를 실제 iOS와 Android 기기에서 테스트합니다.

## 장점

- 안정적인 최소 높이와 현재 보이는 동적 높이를 CSS 단위로 구분할 수 있다.
- Browser chrome과 safe area 때문에 CTA가 가리는 문제를 줄일 수 있다.

## 단점

- `dvh`를 넓게 적용하면 스크롤 중 container 높이가 계속 변해 화면이 흔들릴 수 있다.
- Virtual keyboard는 browser마다 viewport resize 정책이 달라 단위만으로 일관되게 처리되지 않는다.

## 주의사항 / 실무 팁

- Page shell, modal과 bottom sheet의 목적별로 `svh`와 `dvh`를 고른다.
- 주소창 두 상태, 회전, keyboard와 safe-area가 있는 실기기 조합을 테스트한다.
