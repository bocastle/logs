# React Fragment와 DOM 구조 정리

## 핵심 요약

- Fragment로 형제 요소를 묶으면 불필요한 div 없이 table, list, description list의 유효한 DOM 계층을 유지한다.
- 짧은 `<>` 문법에는 key를 줄 수 없어 반복 렌더링에서 각 Fragment 그룹의 정체성을 표현하지 못한다.
- 목록의 여러 열을 한 항목으로 반환할 때는 `<Fragment key={item.id}>`처럼 명시적 문법을 사용한다.

## 개념 설명

React Fragment는 추가 DOM 노드 없이 여러 자식을 묶는 문법이다.

불필요한 wrapper div를 줄여 CSS grid, table, list 구조와 접근성 tree가 의도대로 유지되게 한다.

## 예시

```tsx
return (
  <>
    <dt>Name</dt>
    <dd>{user.name}</dd>
  </>
);
```

description list 안에서 div를 끼워 넣지 않아 HTML 구조가 깨지지 않는다.

## 면접 답변 예시

> 의미 없는 컨테이너가 접근성 트리와 element inspector에 쌓이지 않아 문서 구조를 읽기 쉬워진다. 기존 div를 제거하면 descendant selector와 margin collapse 조건이 달라져 화면 배치가 조용히 바뀔 수 있다. 그룹 자체에 의미나 동작이 있다면 Fragment 대신 section, fieldset, ul 같은 적절한 semantic element를 선택한다.

## 장점

- CSS grid와 flex의 직접 자식 관계가 wrapper에 끊기지 않아 레이아웃 규칙이 실제 콘텐츠 요소에 적용된다.

## 단점

- Fragment는 실제 box를 만들지 않으므로 className, event handler, 배경 스타일이 필요한 wrapper를 대체할 수 없다.

## 주의사항 / 실무 팁

- wrapper 제거 전후의 실제 DOM과 computed layout을 확인해 grid item 및 CSS selector 범위를 비교한다.
