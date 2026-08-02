# React useId 정리

## 핵심 요약

- useId로 label과 input을 연결하면 서버 HTML과 hydration 이후에 같은 접근성 관계가 유지된다.
- useId 값을 목록 key로 쓰면 데이터 정체성을 나타내지 못해 정렬이나 삽입 때 항목 상태가 잘못 보존될 수 있다.
- 컴포넌트마다 useId를 한 번 호출하고 `${id}-hint`, `${id}-error`처럼 의미 있는 suffix로 관련 요소를 묶는다.

## 개념 설명

`useId`는 서버 렌더링과 클라이언트 hydration 사이에서 안정적인 접근성 id를 만들기 위한 React 훅이다.

React 트리의 위치를 기준으로 id를 생성하므로 label, input, aria-describedby 연결에 쓰고 목록 key나 데이터 식별자로 쓰지 않는다.

## 예시

```tsx
function EmailField() {
  const id = useId();
  return <><label htmlFor={id}>Email</label><input id={id} /></>;
}
```

서버 HTML의 id와 클라이언트 id가 맞아 hydration 경고 없이 label 연결이 유지된다.

## 면접 답변 예시

> 여러 React root에 identifierPrefix를 지정하면 마이크로 프런트엔드 사이의 생성 id 충돌을 예방할 수 있다. 생성된 id를 데이터베이스 식별자나 영구 URL에 저장하면 트리 구조 변경 후 같은 대상을 다시 찾을 수 없다. 접근성 테스트에서 label 클릭 시 해당 input에 포커스가 가고 설명 id가 실제 DOM 요소를 가리키는지 검사한다.

## 장점

- 한 필드에서 설명문과 오류문에 suffix를 붙이면 aria-describedby가 충돌 없는 여러 id를 참조할 수 있다.

## 단점

- 서버와 클라이언트의 조건부 트리가 다르면 훅 위치가 달라져 생성 id뿐 아니라 hydration 결과도 어긋난다.

## 주의사항 / 실무 팁

- renderToPipeableStream과 hydrateRoot 양쪽에 동일한 identifierPrefix를 전달해 다중 root의 id 범위를 고정한다.
