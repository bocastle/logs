# React Children API 정리

## 핵심 요약

- Children.toArray는 단일 노드, 배열, null이 섞인 children을 순회 가능한 배열로 정규화한다.
- Children API는 자식 컴포넌트가 렌더할 내부 요소까지 탐색하지 않으므로 깊은 DOM 검사 도구로 쓸 수 없다.
- 복잡한 데이터 전달에는 children 구조 추측보다 Context나 명시적 items prop을 선택한다.

## 개념 설명

React Children API는 `children`을 순회, 개수 확인, 배열 변환할 때 쓰는 보조 도구다.

children은 단일 노드, 배열, null이 될 수 있으므로 `Children.toArray`로 key와 순회를 안정화한다.

## 예시

```tsx
const tabs = Children.toArray(children).filter(isValidElement);
return tabs.map((tab) => cloneElement(tab, { selected }));
```

복합 컴포넌트에서 전달된 자식을 안전하게 배열로 다루지만 과한 cloneElement는 계약을 숨길 수 있다.

## 면접 답변 예시

> Children.count와 only를 사용하면 정확히 하나의 trigger가 필요한 API 제약을 명시적으로 검사할 수 있다. 배열 위치를 의미로 해석하면 조건부 null이나 재정렬 뒤 tab과 panel 연결이 달라질 수 있다. Fragment, 문자열, null, 중첩 배열이 들어오는 테스트로 Children.toArray 이후 순서와 key 동작을 검증한다.

## 장점

- React가 관리하는 key 정보를 보존하며 자식 수와 순서를 다루는 compound component 구현을 단순화한다.

## 단점

- cloneElement로 암묵적 prop을 주입하면 wrapper가 하나 추가되는 순간 계약이 끊기고 타입 추론도 약해진다.

## 주의사항 / 실무 팁

- element prop을 읽기 전에 isValidElement로 확인하고 허용 child type이 있다면 개발 모드 오류를 제공한다.
