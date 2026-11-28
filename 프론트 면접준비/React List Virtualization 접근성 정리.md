# React List Virtualization 접근성 정리

## 핵심 요약

- virtualization은 보이는 row만 DOM에 두어 수만 개 항목의 layout과 paint 비용을 제한한다.
- 포커스된 row가 overscan 밖으로 unmount되면 키보드 초점이 body로 빠져 사용자가 현재 위치를 잃는다.
- 각 row에 실제 전체 index 기반 aria-posinset과 총 item 수 기반 aria-setsize를 설정한다.

## 개념 설명

React List Virtualization 접근성은 화면에 일부 row만 렌더링하면서도 탐색 순서와 전체 목록 맥락을 잃지 않게 하는 문제다.

가상화는 DOM 노드를 재사용하므로 aria-rowcount, aria-posinset, focus 복구, 스크롤 위치 관리가 함께 필요하다.

## 예시

```tsx
<div role="list" aria-setsize={total}>
  {rows.map((row) => <div role="listitem" aria-posinset={row.index + 1}>{row.label}</div>)}
</div>
```

보이는 항목 수와 실제 전체 항목 수가 다르다는 정보를 보조기술에 제공한다.

## 면접 답변 예시

> roving focus와 scroll-to-index를 결합해 키보드 이동 시 필요한 항목을 먼저 렌더링할 수 있다. 재사용 DOM node의 id와 aria label을 갱신하지 않으면 새 데이터가 이전 항목의 접근성 이름을 가진다. 스크린 리더와 키보드 E2E에서 처음, 중간, 마지막 항목 이동 및 데이터 갱신 뒤 위치 복원을 검사한다.

## 장점

- aria-posinset과 aria-setsize를 제공하면 보조기술이 렌더된 일부 항목의 전체 목록 위치를 이해한다.

## 단점

- 화면에 없는 항목을 보조기술이 탐색할 수 있다고 가정하면 전체 목록 검색과 읽기 경험이 불완전해진다.

## 주의사항 / 실무 팁

- 현재 focus item은 overscan에 유지하고 PageDown 및 검색 이동 전에 target index로 스크롤한 뒤 포커스를 준다.
