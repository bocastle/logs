# React Error Boundary 정리

> 요약: **Error Boundary**는 하위 트리에서 발생한 렌더링 오류를 **격리**하고, 앱 전체 다운을 막아 **대체 UI**(fallback)를 보여주는 **클래스 컴포넌트 기반** 보호막입니다. 선언적으로 영역을 감싸 “에러 시 무엇을 보여줄지”를 명시하면, 구체적 처리 흐름은 리액트가 수행합니다.

---

## 1) Error Boundary란?

- **정의**: 자식 컴포넌트의 **렌더링/라이프사이클/생성자** 단계에서 던져진 오류를 포착하여, 해당 서브트리만 **대체 UI**로 전환하는 특수 컴포넌트.
- **목적**: 전체 앱이 하얀 화면으로 무너지는 것을 방지하고, **신뢰성(Resilience)** 과 **UX** 개선.
- **구현 제약**: _클래스 컴포넌트_ 에서만 직접 구현 가능(`getDerivedStateFromError`, `componentDidCatch`). 함수 컴포넌트는 **감싸여 사용**할 수 있음.

### 포착되는 오류

- 자식 트리에서 발생한 **render()**, **constructor**, **라이프사이클**(예: `componentDidMount` 등) 중 throw 된 오류

### 포착되지 않는 오류

- **이벤트 핸들러** 내부 오류(직접 try/catch 필요)
- **비동기 콜백**(Promise/`setTimeout`)에서의 오류(적절한 reject/throw 연결 필요)
- **SSR 렌더링** 단계에서의 오류(서버 프레임워크 레벨 처리)
- **Error Boundary 자신**에서 발생한 오류(바깥 경계가 필요)

---

## 2) 최소 예시

```tsx
import React from "react";

type State = { hasError: boolean };

export class ErrorBoundary extends React.Component<
  React.PropsWithChildren,
  State
> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: unknown): State {
    // 다음 렌더에서 대체 UI 표시
    return { hasError: true };
  }

  componentDidCatch(error: unknown, info: React.ErrorInfo) {
    // 로깅/알림 전송 지점
    console.error("Caught by ErrorBoundary:", error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return <FallbackUI />; // 대체 화면
    }
    return this.props.children;
  }
}

function FallbackUI() {
  return <div>문제가 발생했습니다. 잠시 후 다시 시도해 주세요.</div>;
}

// 사용
// <ErrorBoundary>
//   <FeatureArea />
// </ErrorBoundary>
```

> 함수 컴포넌트 중심 프로젝트에서는 검증된 유틸 컴포넌트(예: 커스텀 래퍼 또는 외부 라이브러리)를 만들어 **경계 배치만** 함수식으로 관리하는 방식을 권장합니다.

---

## 3) 선언형(Declarative) 처리와 유지보수성

- **선언형**: “에러가 나면 _이 영역_ 을 **이 UI** 로 바꿔라”를 **구성**으로 작성(경계 배치).
  - 비즈니스 코드와 **에러 복구 UI**가 분리되어 **관심사 분리**(SoC) 달성.
  - 코드 독해 시 **경계 위치와 Fallback**이 한눈에 보여 가독성·추론성 ↑.
  - 트리 구조를 활용한 **세분화된 격리**(위젯/페이지/라우트 단위)가 쉬움.

---

## 4) 배치 전략(어디에 감쌀까)

- **격리 단위**를 작게: 개별 위젯, 페이지 섹션, 라우트 경계에 배치하여 파급 최소화.
- **라우팅 경계**: 페이지 전환 시 자연스러운 복구/초기화. (React Router/Next.js의 에러 경계 패턴 활용)
- **비동기 섹션**: Suspense 경계와 인접 배치(데이터 로딩 실패 → 에러 경계가 사용자 친화적 메시지/재시도 제공).

---

## 5) 로깅·복구 패턴

- `componentDidCatch`에서 **로깅/모니터링**(Sentry 등) 전송.
- **“재시도” 버튼** 제공: state reset or key 변경으로 **경계 재마운트 → 재시도**.
  ```tsx
  // key를 바꾸면 하위 트리를 재생성하여 복구 시도
  <ErrorBoundary key={retryKey}>
    <ProblematicArea />
  </ErrorBoundary>
  ```
- **사용자 안내**: 연락 채널, 고객센터 링크, 최근 작업 복원 안내.

---

## 6) 비동기/이벤트 오류 다루기

- **이벤트 핸들러**: 직접 try/catch 또는 오류를 **상태로 승격**하여 UI로 처리.
- **Promise 오류**: 렌더 경로로 연결하려면 **throw** 를 트리거하는 래퍼(리소스 패턴) 사용 또는 적절한 **reject 처리 후 경계와 상태 전환**.
- 전역 미처리(rejection) 핸들링(`window.onunhandledrejection`)은 **보조 수단**이지 경계 대체가 아님.

---

## 7) Suspense와의 관계

- **로딩 실패**(데이터 fetch 에러)는 **Error Boundary**가 담당,
- **로딩 중** 상태는 **Suspense**의 `fallback` 이 담당 → 나란히 배치해 역할 분리.
  ```tsx
  <ErrorBoundary>
    <React.Suspense fallback={<Spinner />}>
      <UserPanel />
    </React.Suspense>
  </ErrorBoundary>
  ```

---

## 8) 베스트 프랙티스 체크리스트

- [ ] 위젯/페이지/라우트 경계 등 **다층 경계**로 파급 최소화
- [ ] Fallback에 **재시도/홈 링크** 제공
- [ ] `componentDidCatch` 에서 **에러/스택 로깅**
- [ ] 경계 **자체 오류** 대비 바깥에 상위 경계 추가
- [ ] **테스트**: 의도적으로 throw 하여 Fallback/로그 전송 검증
- [ ] Suspense와 **역할 분리** 설계

---

## 9) 한계와 주의

- **클래스 기반** API 제한(함수형 훅만으로 구현 불가) → 경계 래퍼 재사용.
- **이벤트/비동기/SSR** 등은 자동 포착 X → 별도 경로로 처리 필요.
- 과도한 상위 단일 경계는 **과잉 포착**으로 UX 저하(앱 전체가 동일 에러 화면).

---

## 결론

- Error Boundary는 **UI 안전장치**입니다. 경계를 **적절한 위치**에 배치하고, **로그·복구·안내**를 포함한 Fallback을 설계하면, 부분 실패에도 **전체 앱의 안정성**과 **가독성/유지보수성**을 동시에 확보할 수 있습니다.
