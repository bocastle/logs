# Context Selector 정리

## 핵심 요약
- `Context Selector`는 Context 전체가 아니라 필요한 값만 선택해서 구독하는 접근이다.
- 핵심 목적은 Context 사용 시 불필요한 리렌더링을 줄이는 것이다.
- 면접에서는 `Prop Drilling`, `Context API`, 전역 상태 관리 도구와 연결해서 설명하면 맥락이 잘 잡힌다.
- "Context가 느리다"가 아니라 "값 구조와 구독 범위를 잘못 잡으면 비효율이 생긴다"라고 설명하는 게 더 정확하다.

## 개념 설명
기본적인 Context API는 Provider의 값이 바뀌면 그 Context를 사용하는 컴포넌트들이 다시 렌더링될 수 있다. 값 객체 안에 여러 필드가 섞여 있으면, 실제로는 하나의 필드만 필요한 컴포넌트도 영향을 받을 수 있다.

`Context Selector`는 여기서 필요한 조각만 골라 구독한다는 아이디어다. 예를 들어 사용자 이름만 필요한 컴포넌트는 이름만, 테마만 필요한 컴포넌트는 테마만 선택해서 반응하게 만드는 식이다.

즉, 문제의 본질은 "Context를 쓰면 안 된다"가 아니라 "변경 범위와 구독 범위를 같이 설계해야 한다"는 점이다.

## 예시
예를 들어 다음과 같이 하나의 Context에 여러 값이 있다고 보자.

```tsx
type AppState = {
  theme: "light" | "dark";
  userName: string;
  notifications: number;
};
```

이 상태에서 헤더는 `userName`만, 배지는 `notifications`만 필요할 수 있다. 그런데 Context 전체를 통째로 읽으면 `theme`가 바뀌어도 둘 다 다시 렌더링될 수 있다.

이럴 때는 보통 다음 같은 전략을 쓴다.
- Context를 관심사별로 나눈다
- selector 패턴을 지원하는 라이브러리를 쓴다
- 자주 바뀌는 상태는 Zustand, Redux 같은 별도 상태 저장소로 분리한다

## 면접 답변 예시
- 질문: "Context Selector는 어떤 문제를 해결하나요?"  
  답변: "기본 Context는 Provider 값이 바뀔 때 관련 소비 컴포넌트들이 넓게 다시 렌더링될 수 있는데, Context Selector는 필요한 값만 선택해서 구독하게 해 불필요한 리렌더링을 줄이는 접근입니다. 특히 하나의 Context에 여러 상태를 묶어 둔 화면에서 효과가 큽니다."
- 질문: "그럼 Context Selector만 쓰면 상태 관리 문제가 끝나나요?"  
  답변: "그렇지는 않습니다. 상태가 복잡하거나 비동기 서버 상태가 많으면 Context만으로 관리하기보다 React Query, Zustand, Redux 같은 도구와 역할을 나누는 편이 더 낫습니다. Context Selector는 Context의 구독 범위를 세밀하게 다듬는 방법에 가깝습니다."

## 장점
- 불필요한 리렌더링을 줄이는 데 도움이 된다
- 하나의 큰 Context를 무조건 쪼개지 않아도 되는 여지가 생긴다
- 상태 구독 범위를 의식하게 만들어 구조 설계가 더 명확해진다

## 단점
- 기본 Context API만으로는 바로 쓰기 어려워 추가 패턴이나 라이브러리 이해가 필요할 수 있다
- 잘못 쓰면 오히려 구조가 복잡해져 팀 학습 비용이 늘어난다
- 렌더링 문제의 원인이 Context만은 아닐 수 있어서 과도한 최적화가 될 수 있다

## 주의사항 / 실무 팁
- 먼저 React DevTools Profiler로 실제 병목이 Context 때문인지 확인하는 게 좋다.
- 값 변경 주기가 다른 상태를 하나의 Context에 무조건 몰아넣지 말고, 관심사별 분리와 selector 전략을 같이 검토한다.
- 면접에서는 "Context Selector는 리렌더링 최적화 기법"이라고 요약한 뒤, Context 분리와 상태 관리 도구 선택 기준까지 이어서 말하면 답변이 단단해진다.
