# React useInsertionEffect 정리

## 핵심 요약

- useInsertionEffect에서 스타일 규칙을 넣으면 layout effect의 DOM 측정 전에 CSSOM이 준비되어 측정값이 안정된다.
- 이 훅에서 상태 업데이트를 예약하면 React commit의 삽입 단계와 렌더 단계를 섞어 지원되지 않는 동작에 의존하게 된다.
- useInsertionEffect는 CSS-in-JS 런타임 구현에만 두고 일반 컴포넌트의 동기화에는 effect나 layout effect를 선택한다.

## 개념 설명

`useInsertionEffect`는 CSS-in-JS 라이브러리가 layout effect보다 먼저 스타일을 삽입하도록 마련된 훅이다.

DOM 측정 전에 style tag를 넣어야 하는 경우에만 사용하며 일반 상태 동기화나 데이터 요청에는 맞지 않는다.

## 예시

```tsx
useInsertionEffect(() => {
  sheet.insert(rule);
  return () => sheet.remove(rule);
}, [rule]);
```

스타일 규칙이 layout 계산 전에 들어가 깜빡임과 잘못된 측정을 줄인다.

## 면접 답변 예시

> 스타일 삽입 책임을 라이브러리 계층에 모으면 애플리케이션 effect와 렌더링 순서 경쟁을 피할 수 있다. 큰 stylesheet 생성이나 동기 파싱을 수행하면 브라우저가 paint하기 전의 전체 commit 시간을 막는다. SSR에서 추출한 style tag와 hydration registry가 같은 class 순서를 쓰는지 production 렌더로 확인한다.

## 장점

- CSS-in-JS가 commit마다 필요한 class만 먼저 등록하면 새 컴포넌트가 무스타일 상태로 보이는 순간을 줄일 수 있다.

## 단점

- insertion 단계에는 ref가 아직 연결되지 않을 수 있어 DOM 크기나 노드 존재를 읽는 코드가 불안정하다.

## 주의사항 / 실무 팁

- 동일 규칙을 여러 commit에서 삽입하지 않도록 class hash와 registry를 사용해 작업을 멱등하게 만든다.
