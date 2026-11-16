# React hydration error recovery 정리

## 핵심 요약

- 서버와 첫 클라이언트 렌더를 결정적으로 맞추면 React가 기존 HTML을 재사용해 빠르게 event를 연결한다.
- Date나 Math.random을 render 중 호출하면 서버와 브라우저가 다른 text를 만들어 반복 가능한 mismatch가 생긴다.
- 시간, locale, feature flag 같은 첫 화면 입력은 서버에서 계산해 props로 보내고 hydration 뒤 effect에서만 갱신한다.

## 개념 설명

React hydration error recovery는 서버 HTML과 클라이언트 렌더 결과가 다를 때 오류를 감지하고 가능한 범위에서 복구하는 동작이다.

시간, 랜덤값, 브라우저 전용 상태를 첫 렌더에 섞으면 mismatch가 생기므로 클라이언트 effect 이후에 갱신한다.

## 예시

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
return mounted ? <ClientClock /> : null;
```

서버와 첫 클라이언트 렌더를 맞춘 뒤 브라우저 전용 UI를 표시해 hydration 경고를 줄인다.

## 면접 답변 예시

> onRecoverableError로 복구된 hydration 오류를 수집하면 사용자 화면이 계속 보여도 원인과 발생률을 추적한다. 루트 바깥에서 구조 mismatch가 나면 React가 서버 DOM을 버리고 전체를 다시 렌더링해 초기 입력과 paint를 잃을 수 있다. production SSR HTML을 실제 브라우저에서 hydrate하며 경고와 DOM 교체 여부를 회귀 테스트로 확인한다.

## 장점

- Suspense 경계 안의 mismatch를 격리하면 복구 과정이 페이지 전체를 client render로 바꾸는 범위를 줄일 수 있다.

## 단점

- suppressHydrationWarning을 상위 노드에 넓게 적용하면 실제 attribute와 자식 구조 차이가 진단에서 가려진다.

## 주의사항 / 실무 팁

- hydrateRoot의 onRecoverableError에 component stack과 URL 및 배포 버전을 기록해 원인 컴포넌트를 찾는다.
