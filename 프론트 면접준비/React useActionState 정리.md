# React useActionState 정리

## 핵심 요약

- useActionState는 서버 검증 결과를 같은 폼의 state로 되돌려 필드 오류와 제출 결과를 한 흐름에서 렌더링한다.
- action 함수의 첫 인자인 이전 state를 빼먹으면 FormData 위치를 잘못 해석해 서버에서 필드 값을 읽지 못한다.
- 서버 action에서 FormData를 schema로 파싱하고 `{ fieldErrors, message }`처럼 렌더링 가능한 상태만 반환한다.

## 개념 설명

`useActionState`는 폼 action의 이전 상태와 서버 처리 결과를 연결해 오류와 성공 상태를 화면에 돌려주는 훅이다.

action 함수는 현재 state와 FormData를 받아 다음 state를 반환하고, React는 pending과 결과 상태를 렌더링 흐름에 반영한다.

## 예시

```tsx
const [state, formAction, pending] = useActionState(saveProfile, initialState);
return <form action={formAction}><ErrorList errors={state.errors} /></form>;
```

서버 검증 오류가 같은 폼 상태로 돌아오므로 입력 필드 옆 안내와 재시도를 자연스럽게 연결한다.

## 면접 답변 예시

> isPending을 버튼과 aria-busy에 연결해 FormData 처리 중 중복 제출을 줄이고 진행 상태를 알릴 수 있다. 성공 후 uncontrolled 필드와 action state를 따로 초기화하지 않으면 완료 메시지 옆에 이전 입력이 계속 보인다. 오류 요약은 aria-live로 알리고 각 fieldErrors 항목은 해당 input의 aria-describedby와 연결한다.

## 장점

- formAction을 실제 form action에 연결하면 JavaScript가 준비되기 전에도 브라우저 제출 동작을 활용할 수 있다.

## 단점

- 두 제출이 겹친 상태에서 응답 순서를 고려하지 않으면 늦은 검증 결과가 사용자가 고친 입력의 오류로 남을 수 있다.

## 주의사항 / 실무 팁

- 제출 버튼의 disabled만 믿지 말고 action 자체에 idempotency key를 적용해 재전송의 부작용을 막는다.
