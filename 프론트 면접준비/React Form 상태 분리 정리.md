# React Form 상태 분리 정리

## 핵심 요약

- draft와 useActionState 결과를 분리하면 늦게 온 서버 오류가 사용자가 계속 수정한 입력값을 덮지 않는다.
- 같은 email을 draft와 server state 양쪽에서 canonical 값으로 관리하면 어느 쪽이 최신인지 충돌한다.
- 각 값에 소유자, 갱신 사건, 보존 수명을 적어 input draft와 제출 snapshot 및 응답 상태를 구분한다.

## 개념 설명

React Form 상태 분리는 입력 draft, 검증 오류, 제출 pending, 서버 결과를 한 객체에 뭉치지 않고 역할별로 나누는 설계다.

즉시 반응해야 하는 입력 상태와 서버 왕복으로 바뀌는 action state를 분리하면 race condition과 불필요한 렌더링을 줄인다.

## 예시

```tsx
const [draft, setDraft] = useState(initial);
const [submitState, action] = useActionState(save, idleState);
```

입력 중인 값과 제출 결과가 따로 움직여 서버 오류가 새 draft를 덮어쓰지 않는다.

## 면접 답변 예시

> 필드 오류, 폼 메시지, 성공 결과의 소유자가 나뉘어 각 상태를 초기화하는 조건이 명확해진다. submitting, succeeded, failed boolean을 독립적으로 두면 서로 모순되는 조합이 렌더될 수 있다. 빠른 재입력, 연속 제출, 성공 후 reset, 일부 필드 수정 시나리오를 통합 테스트로 검증한다.

## 장점

- pending 범위를 제출 흐름에만 적용해 키 입력과 로컬 validation은 네트워크 중에도 즉시 반응한다.

## 단점

- 입력 한 번에 모든 서버 오류를 지우면 사용자가 수정하지 않은 다른 필드의 유효한 안내도 사라진다.

## 주의사항 / 실무 팁

- 서버 응답에 제출 version을 포함하거나 action state와 snapshot을 묶어 현재 draft와의 관련성을 확인한다.
