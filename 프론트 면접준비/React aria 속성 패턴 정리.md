# React aria 속성 패턴 정리

## 핵심 요약

- aria-describedby로 입력과 도움말을 연결하면 스크린 리더가 포커스 시 형식과 오류 이유를 함께 읽는다.
- 존재하지 않는 설명 id를 aria-describedby에 남기면 보조기술이 오류 문구를 찾지 못한다.
- useId에서 파생한 hintId와 errorId를 실제 렌더된 요소에만 연결해 참조 무결성을 유지한다.

## 개념 설명

React의 aria 속성 패턴은 DOM attribute 이름을 유지해 보조기술이 읽을 관계와 상태를 JSX에 명확히 적는 방식이다.

`aria-describedby`, `aria-invalid`, `role`은 시각적 상태와 실제 접근성 트리가 일치하도록 상태와 함께 갱신해야 한다.

## 예시

```tsx
<input aria-invalid={Boolean(error)} aria-describedby={error ? errorId : undefined} />
<p id={errorId} role="alert">{error}</p>
```

오류가 있을 때만 설명 id를 연결해 스크린 리더가 현재 입력 문제를 읽을 수 있다.

## 면접 답변 예시

> DOM attribute와 React 상태를 같은 렌더에서 계산하면 시각적 메시지와 접근성 트리의 불일치를 줄인다. 오류가 해결된 뒤에도 aria-invalid가 true로 남으면 사용자는 성공한 입력을 계속 잘못된 필드로 인식한다. Testing Library의 role과 accessible description 쿼리로 화면 문구가 아니라 접근성 트리 결과를 검증한다.

## 장점

- aria-invalid를 실제 검증 state와 동기화하면 색상에 의존하지 않고 현재 필드의 오류 상태를 전달한다.

## 단점

- native button에 임의 role을 덮어쓰면 키보드 동작과 기본 의미를 직접 다시 구현해야 할 수 있다.

## 주의사항 / 실무 팁

- 비동기 검증 결과는 적절한 aria-live 영역에 알리되 키 입력마다 같은 메시지를 반복 방송하지 않는다.
