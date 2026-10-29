# React Transition UX 패턴 정리

## 핵심 요약

- startTransition으로 비싼 결과 갱신을 표시하면 input의 urgent state가 타이핑마다 즉시 commit된다.
- controlled input 값을 transition 안에서만 갱신하면 React가 입력 source of truth를 즉시 맞추지 못한다.
- 입력 문자열은 일반 setState로 갱신하고 그 값에서 파생되는 큰 목록이나 route 전환만 transition으로 분리한다.

## 개념 설명

React Transition UX 패턴은 긴 상태 전환을 긴급 업데이트와 비긴급 업데이트로 나눠 입력 반응성을 지키는 방식이다.

`startTransition`으로 결과 목록 갱신을 낮은 우선순위로 보내고 현재 입력이나 버튼 반응은 즉시 유지한다.

## 예시

```tsx
const [pending, startTransition] = useTransition();
function onChange(value: string) {
  setInput(value);
  startTransition(() => setQuery(value));
}
```

입력값은 바로 바뀌고 비싼 검색 결과 갱신은 pending 상태와 함께 늦춰진다.

## 면접 답변 예시

> useTransition의 isPending으로 기존 결과를 유지한 채 작은 진행 표시를 제공할 수 있다. await 뒤의 setState를 별도 transition으로 감싸지 않는 코드에서는 비동기 후속 업데이트가 의도한 우선순위를 잃을 수 있다. CPU가 느린 환경에서 keystroke latency와 중단된 render 수를 측정해 실제 interaction 개선을 검증한다.

## 장점

- 낮은 우선순위 render는 새 입력이 오면 중단하고 최신 결과로 다시 시작할 수 있어 오래된 화면 작업을 줄인다.

## 단점

- 모든 상태 변경을 비긴급으로 표시하면 선택과 탐색처럼 즉각 보여야 할 피드백까지 늦어진다.

## 주의사항 / 실무 팁

- isPending 표시에는 aria-busy를 연결하되 기존 결과를 흐리게 해 읽지 못하게 만들지는 않는다.
