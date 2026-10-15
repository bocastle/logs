# React Hook 순서 규칙 정리

## 핵심 요약

- 모든 렌더에서 Hook 호출 순서가 같으면 React가 각 state와 effect를 같은 내부 slot에 연결한다.
- 조건부 return 뒤에 Hook을 두면 다음 렌더가 더 많은 slot을 요구해 이전 state가 다른 호출에 매칭된다.
- 모든 Hook을 컴포넌트 최상위에서 호출한 뒤 조건은 effect 본문이나 계산 결과의 사용 지점으로 옮긴다.

## 개념 설명

React Hook 순서 규칙은 모든 렌더에서 훅 호출 순서가 같아야 한다는 제약이다.

React는 hook slot 순서로 상태를 연결하므로 조건문이나 반복문 안에서 훅을 호출하면 이전 상태와 다른 훅이 매칭될 수 있다.

## 예시

```tsx
function Panel({ enabled }: { enabled: boolean }) {
  const [open, setOpen] = useState(false);
  if (!enabled) return null;
  useEffect(() => setOpen(true), []);
}
```

이 예시는 조건부 return 뒤 effect를 호출하므로 enabled 변화 때 훅 순서가 깨질 수 있다.

## 면접 답변 예시

> Custom Hook도 최상위 호출 규칙을 따르면 내부 구현이 바뀌어도 소비 컴포넌트의 slot 순서는 안정적이다. event handler나 일반 함수에서 Hook을 부르면 렌더 dispatcher가 없는 시점에 invalid hook call이 발생한다. eslint-plugin-react-hooks를 CI에 포함하고 early return 경로가 바뀌는 회귀 사례를 렌더 테스트에 둔다.

## 장점

- Rules of Hooks lint가 조건문과 반복문 안의 호출을 찾아 상태가 뒤바뀌는 오류를 실행 전에 막는다.

## 단점

- 배열 길이에 따라 반복문에서 useState를 호출하면 항목 수 변경 때 전체 Hook 순서가 달라진다.

## 주의사항 / 실무 팁

- 항목별 state가 필요하면 반복 호출 대신 각 항목을 안정적인 key를 가진 자식 컴포넌트로 분리한다.
