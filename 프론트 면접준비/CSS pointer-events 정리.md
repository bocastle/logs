# CSS pointer-events 정리

## 핵심 요약

- 장식 오버레이가 클릭을 가로막는 문제를 줄인다.
- 부모에 `pointer-events: none`을 주면 자식 click도 기대와 달라질 수 있다.
- 장식 요소에만 우선 적용한다.

## 개념 설명

`pointer-events`는 요소가 마우스, 터치, 펜의 hit testing 대상이 되는지 조절하는 CSS 속성이다.

`pointer-events: none`을 주면 요소 자체가 click 대상에서 빠지고 아래 요소가 hit testing될 수 있다. 장식 레이어에 적합하지만 상호작용 요소에는 위험하다.

## 예시

```css
.badge-decoration {
  pointer-events: none;
}
```

장식 배지가 카드의 click hit testing을 가로막지 않게 하는 예다.

## 면접 답변 예시

> `pointer-events`는 element가 mouse, pen과 touch hit testing 대상이 될지를 정합니다. 장식 overlay에 `none`을 주면 아래 control이 pointer event를 받을 수 있지만 실제 button을 비활성화하는 수단으로 쓰면 안 됩니다. Pointer hit testing에서 빠져도 keyboard focus와 programmatic activation은 별개라 disabled semantic과 state를 따로 제공해야 합니다. Menu와 tooltip처럼 hover·outside click에 의존하는 layer에는 적용 범위를 좁히고 pointer와 keyboard 경로를 함께 테스트합니다.

## 장점

- 지도나 캔버스 위 레이블의 hit testing 책임을 조절할 수 있다.

## 단점

- 키보드 focus 가능 여부와 pointer hit testing은 별개라 혼란이 생길 수 있다.

## 주의사항 / 실무 팁

- click, hover, keyboard focus 동작을 함께 확인한다.
