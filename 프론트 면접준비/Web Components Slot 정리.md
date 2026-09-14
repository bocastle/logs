# Web Components Slot 정리

## 핵심 요약

- 사용자가 의미 있는 DOM을 유지하면서 컴포넌트 레이아웃에 참여시킬 수 있다.
- slotchange는 배정 노드 자체의 자손 변경까지 알려주지 않는다.
- slot 이름을 공개 컴포넌트 API로 문서화한다.

## 개념 설명

Web Components의 slot은 컴포넌트 사용자가 제공한 light DOM 자식을 shadow tree의 지정된 위치에 투영하는 콘텐츠 합성 지점이다.

이름 없는 slot은 `slot` attribute가 없는 노드를 받고 이름 있는 slot은 같은 이름의 노드를 받으며, 배정 목록 변화는 `slotchange`와 `assignedElements`로 관찰한다.

## 예시

```ts
const shadow = card.attachShadow({ mode: "open" });
shadow.innerHTML = `<header><slot name="title">제목 없음</slot></header><slot></slot>`;
const titleSlot = shadow.querySelector<HTMLSlotElement>('slot[name="title"]')!;
titleSlot.addEventListener("slotchange", () => {
  const [heading] = titleSlot.assignedElements({ flatten: true });
  card.toggleAttribute("has-title", Boolean(heading));
});
```

title slot의 배정 요소가 바뀔 때 assignedElements 결과로 상태를 갱신한다. 배정 노드가 없으면 slot 안의 기본 콘텐츠가 표시된다.

## 면접 답변 예시

> 기본 slot 콘텐츠로 빈 상태를 제공할 수 있다. 투영된 요소의 heading 구조와 접근성 이름을 컴포넌트가 자동 보장하지 않는다. assignedElements 결과의 시맨틱 역할과 heading 순서를 확인한다.

## 장점

- 이름 있는 투영 지점으로 제목과 본문 역할을 구분할 수 있다.

## 단점

- 같은 이름의 slot과 자식이 어긋나면 콘텐츠가 보이지 않을 수 있다.

## 주의사항 / 실무 팁

- 빈 slot, 텍스트 노드, 여러 배정 요소를 모두 테스트한다.
