# CSS overscroll-behavior 정리

## 핵심 요약

- nested scroller와 page scroller의 책임을 나눈다.
- chaining을 막으면 사용자가 page 스크롤을 기대한 지점에서 멈출 수 있다.
- 배경 scroll 누출이 실제 문제인 nested scroller에만 적용한다.

## 개념 설명

CSS `overscroll-behavior`는 scroll container가 경계에 도달했을 때 조상 scroller로 scroll을 이어 보낼지와 브라우저의 boundary effect를 허용할지 정한다.

`auto`는 기본 scroll chaining을 유지하고 `contain`은 현재 scroller에서 chaining을 멈추되 해당 요소의 overscroll effect는 남길 수 있다. `none`은 boundary effect도 억제한다.

## 예시

```css
.command-palette__results {
  overflow-y: auto;
  overscroll-behavior-y: contain;
}
```

커맨드 팔레트 목록 끝에서 wheel을 계속 굴려도 배경 page가 스크롤되지 않게 한다.

## 면접 답변 예시

> `overscroll-behavior`는 nested scroller가 끝에 닿았을 때 scroll을 조상으로 이어 보낼지와 platform boundary effect를 유지할지 정합니다. Modal이나 command palette에서 실제 background scroll 누출이 있을 때 축별 `contain`을 사용하고 page 전체에 무조건 적용하지 않겠습니다. `none`은 chaining뿐 아니라 pull-to-refresh 같은 boundary feedback도 막을 수 있어 목적이 명확할 때만 선택합니다. Scroll 가능한 높이를 가진 상태에서 touch, trackpad와 wheel로 contain과 none의 차이를 실제 device에서 확인합니다.

## 장점

- 모달·drawer 내부 스크롤이 배경으로 튀는 문제를 줄인다.

## 단점

- `none`은 플랫폼 boundary feedback까지 제거해 상태를 알기 어렵게 만든다.

## 주의사항 / 실무 팁

- touch·trackpad·wheel에서 경계 동작을 확인한다.
