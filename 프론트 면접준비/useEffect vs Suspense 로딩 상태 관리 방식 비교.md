## 🔄 useEffect vs 🌀 Suspense: 로딩 상태 관리 방식 비교

### 🧠 전통적인 방식 - `useEffect + loading state`

`useEffect()`와 별도의 `isLoading` 상태를 활용하여 로딩 상태를 수동으로 제어합니다.

```tsx
const [isLoading, setIsLoading] = useState(true);

useEffect(() => {
  fetchData().then(() => {
    setIsLoading(false);
  });
}, []);
```

- 데이터를 요청하기 전: `isLoading = true`
- 데이터를 모두 받아온 후: `isLoading = false`
- 조건부 렌더링을 통해 로딩 UI 제어

❗ **단점**

- 여러 비동기 작업이 중첩되면 상태 관리 및 조건부 렌더링이 복잡해짐
- 매번 로딩 상태를 직접 제어해야 하므로 반복 코드가 많아질 수 있음

---

### 🌀 선언형 방식 - `Suspense`

React의 `Suspense`는 **데이터 준비가 완료될 때까지** `fallback` UI를 보여주는 방식입니다.

```tsx
<Suspense fallback={<LoadingSpinner />}>
  <SomeComponent />
</Suspense>
```

- 로딩 중에는 `fallback`으로 지정한 컴포넌트만 보여짐
- 데이터가 준비되면 `Suspense` 내부 컴포넌트가 자연스럽게 렌더링됨

✅ **장점**

- 선언형으로 로딩 상태를 관리 → 코드 간결해짐
- `React.lazy()` 및 `데이터 패칭 라이브러리`와 함께 사용 가능
- 유지보수가 쉬움

---

### ⚠️ Suspense의 단점은 무엇일까요? 🤔

- **트리 구조가 복잡할 경우**, 여러 `Suspense`가 중첩되어 각기 다른 타이밍으로 `fallback`이 렌더링 → 사용자 경험 저하
- 데이터 흐름을 정교하게 설계하지 않으면 불안정한 렌더링 발생
- **Promise 기반**만 지원 → 일반적인 `fetch()`에는 바로 사용 불가  
  → `React Query`, `Relay`, `SWR`, `React Cache` 등과 함께 사용해야 함

---

### ✅ 결론

| 항목             | useEffect 방식               | Suspense 방식         |
| ---------------- | ---------------------------- | --------------------- |
| 로딩 상태 관리   | 수동 상태 관리 (`isLoading`) | 선언형 (`fallback`)   |
| 코드 간결성      | 반복 코드 많음               | 코드 간결             |
| 비동기 처리 방식 | 어떤 비동기든 사용 가능      | Promise 기반만 지원   |
| 설계 난이도      | 단순                         | 트리 구조 고려 필요   |
| 적합한 상황      | 간단한 요청                  | 대규모 트리/복잡한 앱 |

📌 **Tip:** 단순한 요청에는 `useEffect`,  
복잡하거나 선언형 방식이 유리한 상황에는 `Suspense`를 선택하세요.
