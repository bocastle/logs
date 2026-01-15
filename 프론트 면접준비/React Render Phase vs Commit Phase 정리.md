# React Render Phase vs Commit Phase 정리 (React 18 기준)

---

## 1) 각 단계의 역할

### Render Phase — “무엇이 바뀌는가”를 **계산**

- **입력**: 최신 props, state, context, 업데이트 큐
- **작업**:
  - 컴포넌트 함수 실행(VDOM/Fiber 산출)
  - 자식 트리 재귀(리컨실리에이션: 추가/삭제/이동/갱신 결정)
  - 메모이제이션/`shouldComponentUpdate`/`React.memo` 판정
  - 우선순위 스케줄링(Concurrent Rendering): **중단/재시작/폐기 가능**
- **특성**:
  - **DOM 변경 없음**
  - 순수 계산이어야 하므로 **부작용 금지** (네트워크, 구독, 타이머 등)

### Commit Phase — 계산 결과를 **DOM에 반영**

- **입력**: render 결과로 준비된 변경 리스트
- **작업**(세 소단계):
  1. **Before mutation**: DOM 변경 전 스냅샷(클래스 `getSnapshotBeforeUpdate` 등)
  2. **Mutation**: 실제 **DOM/Ref** 변경, 인스턴스 마운트/언마운트
  3. **Layout/Passive Effects**:
     - **`useLayoutEffect` / 클래스 `componentDidMount/Update`**: **동기**, DOM 적용 직후 실행
     - **`useEffect`(passive)**: **화면 페인트 후 비동기**로 실행/클린업
- **특성**:
  - **원자적 적용(atomic)**: 해당 커밋 단위로 일관된 UI 보장
  - 부작용 배치/클린업 타이밍이 명확

---

## 2) 어떤 코드가 어디서 실행되는가

| 범주                                                               | Render Phase | Commit Phase             |
| ------------------------------------------------------------------ | ------------ | ------------------------ |
| 함수 컴포넌트 본문                                                 | ✔︎           |                          |
| `useMemo` 계산                                                     | ✔︎           |                          |
| `useCallback` 생성                                                 | ✔︎           |                          |
| `useRef` 읽기/쓰기(렌더 중 값 읽기는 가능하나 DOM ref는 아직 미결) | ✔︎\*         |                          |
| DOM 변경/Ref 연결                                                  |              | ✔︎(Mutation)             |
| `useLayoutEffect`(setup/cleanup)                                   |              | ✔︎(Mutation 직후 동기)   |
| `useEffect`(setup/cleanup)                                         |              | ✔︎(페인트 **후** 비동기) |
| 클래스 `render()`                                                  | ✔︎           |                          |
| 클래스 `getSnapshotBeforeUpdate()`                                 |              | ✔︎(mutation 전)          |
| 클래스 `componentDidMount/Update/WillUnmount`                      |              | ✔︎                       |

> \*렌더 중 `ref.current`를 읽는 것은 **이전 커밋 상태**를 의미합니다. 새 DOM은 아직 없습니다.

---

## 3) 동기화(스케줄링) 관점의 특징

1. **단계적 진행**

- Render가 끝나도 **즉시 Commit하지 않을 수 있음**. 더 높은 우선순위(입력/타이핑 등)가 오면 먼저 처리 후 커밋.
- Render 결과는 **폐기 가능**: 트랜지션/동시성에서 이전 계산이 버려질 수 있습니다.

2. **일관 커밋(atomic commit)**

- Commit은 **중단되지 않고** 한 번의 스텝으로 적용됩니다. 커밋 중간의 **찢김(tearing)** 을 방지합니다.

3. **우선순위/동시성**

- React 18: `startTransition`, `useTransition`, `useDeferredValue`로 **긴급/비긴급**을 분리.
- 긴급 업데이트는 **선행**, 비긴급은 **중단 재개 가능**. 사용자 입력 체감 개선.

4. **배칭(Batching)**

- 동일 이벤트 루프 틱 내 상태 업데이트는 **자동 배치**되어 render/commit 횟수를 줄입니다(React 18 전역 배칭).

---

## 4) 효과(Effects) 타이밍 요약

- **`useLayoutEffect`**: DOM이 갱신된 **직후, 페인트 이전** 동기 실행. 레이아웃 읽기/즉시 측정/스크롤 위치 보정 등 **레이아웃 민감 작업**에 사용(지나치면 **페인트 지연** 유발).
- **`useEffect`**: **페인트 이후** 비동기. 구독/네트워크/타이머 등 **UI와 느슨히 연관**된 부작용.

```jsx
useLayoutEffect(() => {
  const rect = el.current.getBoundingClientRect(); // 최신 레이아웃
  positionTooltip(rect);
}, [deps]);

useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id); // cleanup
}, []);
```

---

## 5) 개발 모드(Strict Mode) 주의

- 일부 컴포넌트의 **렌더 함수/이펙트 setup**이 **두 번 호출**되어 **순수성 검증**을 돕습니다(실제 프로덕션 커밋은 1회).
- **렌더에서 부작용 금지** 규칙을 지키면 안전합니다.

---

## 6) 실전 가이드

- **렌더 = 순수 함수 원칙**: I/O, 구독, DOM 접근을 넣지 마십시오.
- **레이아웃 민감 로직**은 `useLayoutEffect`, 일반 부작용은 `useEffect`.
- **트랜지션**로 비긴급 UI 갱신을 감싸 입력 지연을 줄이십시오.
- **메모이제이션**(`React.memo`/`useMemo`/`useCallback`)으로 불필요한 render 비용을 줄이되 **핫스팟 위주**로.
- **프로파일링**: React DevTools Profiler로 커밋 횟수/시간을 측정, 병목을 확인.
- **서버 컴포넌트/Suspense**: 대기 구간을 분리하고 **부분 커밋**(스트리밍)으로 UX 개선.

---

## 7) 요약

- **Render Phase**: 가상 트리 계산(중단·재시작·폐기 가능), DOM 미변경.
- **Commit Phase**: DOM/Ref 적용과 이펙트 실행(원자적).
- 두 단계는 **우선순위 기반으로 비동기적으로 동기화**되며, 최종 커밋은 **일관된 화면**을 보장합니다.
