# Canvas 접근성 대안 정리

## 핵심 요약

- 스크린 리더 사용자가 시각화 데이터를 읽는다.
- canvas에 `aria-label`만 추가해서는 여러 data point와 control을 전달할 수 없다.
- 시각화를 만들기 전에 텍스트·table 정보 구조를 정의한다.

## 개념 설명

Canvas 접근성 대안은 pixel로 그린 chart·diagram·game 정보와 상호작용을 DOM text·table·control·description으로 동등하게 제공하는 설계다.

`canvas`는 그림 내부 shape를 자동으로 accessibility tree에 노출하지 않는다. 요약·데이터 table·키보드 컨트롤을 adjacent DOM 또는 fallback content로 유지하고 canvas와 같은 state model에 연결한다.

## 예시

```html
<canvas aria-describedby="sales-summary"></canvas>
<p id="sales-summary">매출은 1월 120, 2월 145, 3월 172입니다.</p>
<table>…월별 데이터…</table>
```

시각 chart는 canvas에 그리되 요약과 같은 원본 데이터 table을 DOM에 제공한다.

## 면접 답변 예시

> Canvas rendering 실패와 관계없이 핵심 정보를 유지한다. canvas 내 pointer 영역만 구현하면 keyboard focus가 없다. 키보드·스크린 리더·200% zoom·high contrast에서 완료 흐름을 테스트한다.

## 장점

- 키보드 사용자가 필터·선택·zoom 기능을 사용한다.

## 단점

- DOM 대안과 canvas가 다른 state를 보이면 정보가 불일치한다.

## 주의사항 / 실무 팁

- DOM과 canvas가 같은 data/state source를 사용하게 한다.
