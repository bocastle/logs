# React Composition over Inheritance 정리

## 핵심 요약

- composition으로 Dialog의 구조와 children 콘텐츠를 나누면 상속 override 없이 화면 변형을 조립할 수 있다.
- 모든 영역을 임의 children으로 열어 두면 필수 제목과 action 순서를 보장하기 어려워진다.
- 변경 가능성이 실제로 다른 영역만 named slot으로 만들고 고정 구조는 컴포넌트 내부에 유지한다.

## 개념 설명

React Composition over Inheritance는 상속 계층보다 props, children, slot 조합으로 UI 변형을 만드는 원칙이다.

컴포넌트를 작은 역할로 나누고 합성하면 상태 경계와 테스트 범위가 명확해진다.

## 예시

```tsx
<Dialog title="삭제 확인">
  <DeleteConfirmActions />
</Dialog>
```

Dialog는 구조와 접근성 역할을 맡고 구체적인 액션은 children으로 전달된다.

## 면접 답변 예시

> 작은 컴포넌트를 props로 결합해 기반 class 내부 구현에 의존하지 않고 기능을 독립적으로 교체한다. 상태 소유자를 잘못 나누면 합성된 자식끼리 같은 selection을 맞추기 위해 callback이 여러 층을 오간다. Dialog처럼 의미가 강한 primitive는 composition 자유도와 별개로 title 및 focus 계약을 내부에서 강제한다.

## 장점

- 명시적 header, body, actions slot은 각 영역의 소유 책임과 테스트 대상을 작게 유지한다.

## 단점

- slot prop이 과도하게 많아지면 한 화면을 구성하려고 긴 plumbing을 반복해 단순한 전용 컴포넌트보다 복잡해진다.

## 주의사항 / 실무 팁

- 공유 상태는 합성 지점의 가장 가까운 소유자에 두고 하위 요소에는 값과 의도 기반 handler를 전달한다.
