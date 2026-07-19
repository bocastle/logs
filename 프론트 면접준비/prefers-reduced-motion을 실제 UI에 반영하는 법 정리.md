# prefers-reduced-motion을 실제 UI에 반영하는 법 정리

## 핵심 요약

- `prefers-reduced-motion`은 사용자가 불필요한 움직임을 줄이도록 운영체제에 설정한 선호를 전달한다.
- 큰 위치 이동, parallax, 반복·자동 재생부터 제거하거나 즉시 상태 전환으로 바꾼다.
- 모션만 없애고 loading·focus·success 같은 상태 단서까지 사라지지 않게 한다.

## 개념 설명

Reduced motion 대응은 모든 애니메이션 duration을 임의로 줄이는 작업이 아니다. 움직임에 민감한 사용자에게 불편을 줄 수 있는 큰 확대·이동, parallax와 반복 효과를 없애고 상태 변화는 색상, 아이콘이나 즉시 전환으로 계속 전달한다.

CSS는 media query로 분기하고 JavaScript animation, canvas와 영상은 `matchMedia()` 결과를 사용한다. 앱 실행 중 시스템 설정 변경을 반영해야 한다면 change event를 구독하되 listener cleanup도 포함한다. 사용자 계정에 별도 설정이 있다면 OS 선호와의 우선순위를 명확히 한다.

## 예시

```css
@media (prefers-reduced-motion: reduce) {
  .route-transition { animation: none; }
  html { scroll-behavior: auto; }
}
```

라우트 전환과 부드러운 스크롤을 없애도 새 화면의 제목으로 포커스를 옮기고 선택 상태를 표시하는 동작은 유지한다.

## 면접 답변 예시

> `prefers-reduced-motion`은 사용자가 운영체제에서 불필요한 움직임을 줄이도록 설정한 선호입니다. 큰 위치 이동과 parallax, 반복 애니메이션을 먼저 없애고 상태 변화 자체는 아이콘이나 즉시 전환으로 남기겠습니다. CSS뿐 아니라 JavaScript animation, canvas와 자동 재생 영상도 같은 정책을 사용해야 합니다. 실행 중 설정 변경을 지원한다면 `matchMedia` change를 구독하고 reduced 모드의 focus·loading·success 흐름을 따로 테스트합니다.

## 장점

- 움직임에 민감한 사용자의 불편을 줄이면서도 상태 변화의 의미를 유지할 수 있다.
- CSS와 JavaScript 모션을 하나의 사용자 선호에 맞춰 운영할 수 있다.

## 단점

- Duration만 지나치게 줄이면 전환이 오히려 갑작스럽고 불편할 수 있다.
- CSS만 분기하고 canvas·영상·animation library를 놓치면 화면마다 경험이 달라진다.

## 주의사항 / 실무 팁

- Reduced 모드에서 focus, loading, success와 오류 상태가 모션 없이도 구분되는지 테스트한다.
- 사용자 앱 설정과 OS 선호가 함께 있을 때의 우선순위와 실시간 변경 listener를 검증한다.
