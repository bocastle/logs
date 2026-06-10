# CSS Anchor Positioning 정리

## 핵심 요약

- Anchor Positioning은 요소를 다른 요소의 위치/영역에 상대적으로 정렬하기 위한 CSS 기능이다.
- `position-anchor`, `anchor()` 함수, `position-try-fallbacks` 같은 개념으로 툴팁/팝오버 배치를 선언적으로 처리한다.
- 절차형 위치 계산 자바스크립트보다 렌더링 성능과 접근성 제어가 좋아, 최근 브라우저 인터랙션에서 선호도가 높다.
- 면접 포인트는 `anchor`의 대상 지정, fallback 순서, 오버플로우 제어, 브라우저 지원 범위다.

## 개념 설명

### 1) Anchor 대상 지정

CSS Anchor Positioning은 기준이 되는 요소를 `anchor-name`으로 이름 붙인다.

```css
.anchor-target {
  anchor-name: --btn;
}

.tooltip {
  position: absolute;
  position-anchor: --btn;
  inset: auto;
}
```

- `anchor-name`은 보통 레이아웃에서 기준이 되는 버튼, 아이콘, 입력 필드 등에 붙인다.
- `position-anchor`로 툴팁이 어느 대상을 기준으로 배치되는지 명시한다.

### 2) `anchor()` 함수로 좌표 계산

```css
.tooltip {
  left: anchor(left);
  top: calc(anchor(bottom) + 8px);
}
```

- `anchor()`는 기준 요소의 경계값을 읽어 정렬 지점으로 사용한다.
- 기존 `transform`/`translate` 조합보다 의도 전달이 명확하다.

### 3) fallback 전략

화면 밖으로 벗어나는 경우를 위해 폴백 위치를 선언한다.

```css
.popover {
  inset: auto;
  margin: 0;

  position-try-fallbacks:
    --left,
    --right,
    --above;
}

@position-try --left {
  left: anchor(left) - 240px;
}

@position-try --right {
  left: anchor(right) + 8px;
}
```

- 면접에서는 “항상 중앙 정렬” 같은 고정값보다 **조건 분기 fallback**를 말할 수 있어야 한다.

## 사용 예시

- 드롭다운 메뉴
- 말풍선형 툴팁
- 입력 필드 에러 메시지
- 캘린더/팝오버

### 예시 코드

```html
<button class="info" id="help">도움말</button>
<div class="tip">추가 설명</div>
```

```css
#help {
  anchor-name: --help-btn;
}

.tip {
  position: absolute;
  position-anchor: --help-btn;
  position-area: bottom;
  translate: -50% 0;
  margin: 6px;
}
```

## 면접 답변 예시

"Anchor Positioning은 요소를 타겟 요소 기준으로 배치할 때, JS로 좌표 계산을 직접 하지 않고 CSS 선언으로 의도를 표현하는 방식입니다. 특히 툴팁이나 팝오버처럼 화면 경계를 넘기기 쉬운 UI에서 `position-try-fallbacks`로 대체 위치를 미리 지정하면 UX가 끊기지 않고 안정적으로 동작합니다. 다만 브라우저 지원을 확인하고, 대체 구현(오버플로우 감시 JS 또는 Popover API 조합)도 준비해야 실서비스 리스크를 낮출 수 있습니다."

## 장점

- 레이아웃 계산 코드 감소
- 툴팁/팝오버 배치 규칙을 CSS에서 일관되게 관리
- 반응형 레이아웃에서 오버플로우 케이스 대응이 쉬움

## 단점

- 브라우저 지원이 완전하지 않을 수 있음
- 고급 fallback 로직을 모르면 의도 전달이 오해로 이어질 수 있음
- 일부 레거시 UI와 혼용 시 테스트 케이스가 늘어남

## 주의사항 / 실무 팁

- 툴팁의 기본 접근성 순서를 깨지 않게 `aria-describedby`를 함께 사용한다.
- 포지셔닝 충돌이 나는 경우에는 `overflow`, `z-index`, stacking context를 먼저 점검한다.
- 브라우저 지원이 모호한 기능은 폴리필/대체 로직을 문서화한다.
