# React state colocation 정리

## 핵심 요약

- 드롭다운 open state를 소유 컴포넌트에 두면 토글할 때 페이지의 넓은 subtree가 다시 렌더링되지 않는다.
- 두 형제가 같은 값을 독립 state로 복제하면 한쪽 변경 뒤 서로 다른 선택을 표시할 수 있다.
- state는 가장 낮은 공통 소유자에서 시작하고 실제로 둘 이상의 소비자가 필요할 때만 한 단계씩 lift up한다.

## 개념 설명

React state colocation은 상태를 실제로 읽고 변경하는 가장 가까운 컴포넌트에 두는 설계 원칙이다.

상태를 불필요하게 위로 올리면 부모 변경이 넓은 subtree 재렌더링으로 이어지므로 공유가 필요한 값만 lift up한다.

## 예시

```tsx
function SearchBox() {
  const [open, setOpen] = useState(false);
  return <Combobox open={open} onOpenChange={setOpen} />;
}
```

드롭다운 열림 상태를 검색 박스 안에 두면 페이지 전체가 그 상태를 알 필요가 없다.

## 면접 답변 예시

> 부모가 알 필요 없는 임시 값을 숨겨 prop drilling과 공유 상태의 우발적 의존을 줄인다. 서버 데이터나 URL로 복원해야 하는 값을 화면 state에만 가두면 새로고침과 공유 경로가 끊긴다. React DevTools로 state 변경 시 렌더 범위를 확인해 colocation이 소비자 경계와 일치하는지 검증한다.

## 장점

- 기능을 삭제하거나 이동할 때 관련 state와 handler가 같은 경계에 있어 변경 범위를 찾기 쉽다.

## 단점

- 조건부 렌더링으로 로컬 소유자가 unmount되면 사용자가 돌아왔을 때 보존해야 할 draft도 사라질 수 있다.

## 주의사항 / 실무 팁

- unmount 후에도 유지할 값인지 먼저 구분하고 필요하면 안정적인 keyed 부모나 외부 저장소에 수명을 맡긴다.
