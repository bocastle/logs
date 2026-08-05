# useImperativeHandle 정리

## 핵심 요약

- useImperativeHandle을 쓰면 ref 공개 범위가 focus 같은 의도된 명령으로 좁아져 커스텀 입력 컴포넌트의 캡슐화가 유지된다.
- useImperativeHandle에 너무 많은 메서드를 넣으면 자식 상태를 외부에서 조종하는 우회 API가 되어 데이터 흐름이 흐려진다.
- handle에는 focus, clear, measure처럼 외부 명령이 꼭 필요한 최소 메서드만 두고 상태 값은 props callback으로 전달한다.

## 개념 설명

`useImperativeHandle`은 부모가 ref로 볼 수 있는 명령형 API를 자식 컴포넌트가 직접 정의하게 해 DOM 전체 대신 제한된 handle만 노출하는 React 훅이다.

`forwardRef`로 받은 ref에 `useImperativeHandle`을 연결하고 focus, reset, scrollToError처럼 선언형 props만으로 표현하기 어려운 동작을 작은 객체로 공개한다.

## 예시

```tsx
type TextFieldHandle = { focus: () => void };
const TextField = forwardRef<TextFieldHandle, Props>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
  }), []);
  return <input ref={inputRef} {...props} />;
});
```

부모는 `ref.current?.focus()`만 호출할 수 있고 내부 input DOM 구조나 임시 상태에는 직접 접근하지 못한다.

## 면접 답변 예시

> 폼 오류 위치 이동이나 모달 초기 포커스처럼 접근성에 필요한 명령형 동작을 선언형 렌더링 모델과 분리해 둘 수 있다. 단순 값 전달까지 ref 명령으로 처리하면 props와 state로 표현 가능한 관계가 테스트하기 어려운 절차 코드로 바뀐다. 키보드 이동, 오류 안내, 모달 open 경로에서 ref 호출 시점이 commit 이후인지 확인해 null handle 접근을 피한다.

## 장점

- forwardRef와 함께 handle 타입을 명시하면 부모가 호출 가능한 메서드를 컴파일 단계에서 확인할 수 있다.

## 단점

- dependency 배열을 잘못 두면 handle이 오래된 props나 callback을 잡아 focus 이후 동작이 현재 렌더와 달라질 수 있다.

## 주의사항 / 실무 팁

- forwardRef 컴포넌트의 공개 handle 타입을 별도 export해 소비자가 DOM 노드가 아니라 제한된 ref 계약을 보게 한다.
