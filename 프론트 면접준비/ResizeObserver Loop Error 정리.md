# ResizeObserver Loop Error 정리

## 핵심 요약

- 반응형 컴포넌트의 실제 컨테이너 크기를 관찰할 수 있다.
- 콜백에서 크기를 바꾸면 같은 프레임에 루프가 생길 수 있다.
- 측정은 observer 콜백에서, 쓰기는 `requestAnimationFrame`에서 처리한다.

## 개념 설명

ResizeObserver loop error는 크기 변화를 관찰하는 콜백 안에서 다시 크기를 바꿔 같은 프레임에 관찰과 레이아웃이 반복될 때 생기는 경고다.

`ResizeObserver` 콜백은 측정 결과를 읽는 데 집중하고, DOM 쓰기는 `requestAnimationFrame`으로 다음 프레임에 넘기면 loop 위험을 줄일 수 있다.

## 예시

```js
let frame = 0;
const observer = new ResizeObserver((entries) => {
  const width = Math.round(entries[0].contentRect.width);
  cancelAnimationFrame(frame);
  frame = requestAnimationFrame(() => {
    const next = `${width}px`;
    if (panel.style.getPropertyValue("--measured-width") !== next) {
      panel.style.setProperty("--measured-width", next);
    }
  });
});
```

콜백에서 바로 쓰지 않고 다음 프레임으로 미루며 같은 값은 다시 쓰지 않는다. 이 custom property가 관찰 대상의 폭 자체를 다시 바꾸는 데 쓰이면 프레임 간 feedback loop가 생길 수 있으므로 용도를 분리해야 한다.

## 면접 답변 예시

> ResizeObserver loop error는 size callback에서 다시 layout size를 바꿔 같은 frame 안에 관찰과 변경이 반복될 때 발생합니다. 측정은 callback에서 하고 DOM write는 다음 animation frame으로 넘기며 실제 값이 바뀐 경우에만 반영하겠습니다. 다만 rAF는 같은 frame의 loop만 끊을 뿐 측정한 폭이 다시 자기 폭을 바꾸면 frame 간 진동이 계속될 수 있어 feedback 구조 자체를 없애야 합니다. 단순 responsive style이라면 JavaScript 측정보다 container query를 먼저 검토합니다.

## 장점

- loop 원인을 콜백 내부 DOM 쓰기로 좁히기 쉽다.

## 단점

- 관찰 대상이 많으면 callback 비용이 커진다.

## 주의사항 / 실무 팁

- 관찰 대상 수를 필요한 컨테이너로 제한한다.
