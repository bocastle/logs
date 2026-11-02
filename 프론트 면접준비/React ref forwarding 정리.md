# React ref forwarding 정리

## 핵심 요약

- forwardRef를 사용하면 Form 라이브러리와 부모 컴포넌트가 커스텀 입력의 실제 focus target에 접근한다.
- 내부 DOM 노드를 그대로 노출하면 마크업을 input에서 contenteditable로 바꾸는 수정이 외부 API의 breaking change가 된다.
- 부모가 실제로 필요한 focus나 measure만 handle로 노출하고 내부 node 접근을 기본 계약으로 만들지 않는다.

## 개념 설명

React ref forwarding은 부모가 자식 컴포넌트 내부 DOM이나 imperative handle에 접근하도록 ref를 전달하는 패턴이다.

`forwardRef`와 `useImperativeHandle`로 공개할 동작을 제한해 포커스 이동 같은 명령형 요구를 처리한다.

## 예시

```tsx
const TextInput = forwardRef<HTMLInputElement, Props>((props, ref) => (
  <input ref={ref} {...props} />
));
```

부모는 커스텀 입력 컴포넌트라도 실제 input에 focus를 줄 수 있다.

## 면접 답변 예시

> ref 타입을 public props와 함께 선언하면 wrapper를 거쳐도 HTMLInputElement 계약이 소비자에게 보존된다. generic 컴포넌트를 forwardRef로 감싸는 과정에서 타입 매개변수가 넓어져 item과 handler의 상관관계를 잃을 수 있다. 지원 React 버전에서 ref prop 패턴과 forwardRef 요구를 확인하고 focus 및 unmount 동작을 통합 테스트한다.

## 장점

- useImperativeHandle과 결합해 DOM 전체 대신 select, focus 같은 제한된 명령만 공개할 수 있다.

## 단점

- 렌더마다 새 callback ref를 합성하면 이전 ref에 null을 주고 다시 연결하는 불필요한 detach가 발생한다.

## 주의사항 / 실무 팁

- 여러 ref를 연결해야 하면 안정적인 mergeRefs helper를 사용하고 callback의 null lifecycle도 처리한다.
