# React controlled async form 정리

## 핵심 요약

- controlled input은 현재 draft를 즉시 state에 반영해 글자 수, 로컬 형식 오류, 제출 가능 여부를 함께 계산한다.
- 상위 폼 전체가 키 입력마다 비싼 계산을 실행하면 controlled 필드의 타이핑이 눈에 띄게 지연된다.
- 자주 바뀌는 필드 state를 작은 컴포넌트에 두고 비싼 미리보기는 deferred value로 분리한다.

## 개념 설명

controlled async form은 입력값을 React 상태로 관리하면서 비동기 검증이나 저장 상태를 함께 다루는 폼이다.

입력 onChange는 즉시 draft를 갱신하고 서버 검증은 debounce, abort, field error state로 분리한다.

## 예시

```tsx
const [value, setValue] = useState("");
const [error, setError] = useState<string | null>(null);
<input value={value} onChange={(event) => setValue(event.target.value)} aria-invalid={!!error} />
```

입력 반응성과 서버 오류 표시를 분리해야 타이핑 지연과 오래된 오류 노출을 줄인다.

## 면접 답변 예시

> 제출 시점의 값 snapshot을 사용하면 요청 중 계속 타이핑해도 어떤 draft를 검증했는지 구분된다. pending 동안 필드를 무조건 잠그면 네트워크가 느릴 때 사용자가 오타를 바로 고치지 못한다. aria-busy는 검증 영역에, aria-invalid는 응답이 현재 input value와 일치할 때만 설정한다.

## 장점

- 비동기 검증을 별도 상태로 두면 입력 반응성과 서버 확인 진행 상태를 서로 다른 우선순위로 렌더링할 수 있다.

## 단점

- 이전 값의 검증 응답이 늦게 도착하면 새 값 아래에 오래된 aria-invalid와 오류 메시지가 표시될 수 있다.

## 주의사항 / 실무 팁

- 서버 검증은 debounce한 뒤 요청별 AbortController나 sequence를 사용해 최신 값의 결과만 적용한다.
