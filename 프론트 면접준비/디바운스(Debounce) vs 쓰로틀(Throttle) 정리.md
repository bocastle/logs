# 디바운스(Debounce) vs 쓰로틀(Throttle) 정리

> 이벤트가 **빈번히 발생**할 때 핸들러 실행을 **제어**하여 성능을 개선하는 기법입니다.  
> 디바운스는 **연속 입력이 멈춘 뒤** 실행, 쓰로틀은 **일정 간격마다** 실행합니다.

---

## 1) 개념 요약

### 디바운스(Debounce)

- 연속 이벤트가 발생하는 동안 **타이머를 계속 리셋**하고, **마지막 이벤트 후 대기 시간**이 지나면 1회 실행.
- 사용 예: 실시간 검색 입력, 자동 저장, 창 크기 변경 후 레이아웃 계산.

```
시간:  |----이벤트 연속----|    (정지)    → 실행
동작:  [타이머 리셋 반복   ]----대기 T----→ handler()
```

### 쓰로틀(Throttle)

- **고정 간격 T** 내에는 **최대 1회**만 실행(선두 실행 또는 후행 실행 선택 가능).
- 사용 예: 스크롤/드래그 위치 추적, 무한 스크롤 트리거, 윈도우 스크롤에 따른 헤더 고정.

```
시간 구간(너비 T): |---구간1---|---구간2---|---구간3---|
선두 실행(leading):   H                H
후행 실행(trailing):        H                H
```

---

## 2) 자바스크립트 기본 구현

### 2.1 디바운스(leading/trailing 옵션)

```js
function debounce(fn, wait, { leading = false, trailing = true } = {}) {
  let timer = null;
  let lastArgs,
    lastThis,
    invoked = false;

  const invoke = () => {
    const args = lastArgs;
    const ctx = lastThis;
    lastArgs = lastThis = null;
    invoked = false;
    fn.apply(ctx, args);
  };

  return function (...args) {
    lastArgs = args;
    lastThis = this;

    if (timer) clearTimeout(timer);

    if (leading && !invoked) {
      fn.apply(this, args);
      invoked = true;
    }

    if (trailing) {
      timer = setTimeout(invoke, wait);
    }
  };
}
```

### 2.2 쓰로틀(leading/trailing 선택)

```js
function throttle(fn, wait, { leading = true, trailing = true } = {}) {
  let lastCall = 0;
  let timer = null;
  let lastArgs, lastThis;

  const invoke = () => {
    lastCall = Date.now();
    timer = null;
    fn.apply(lastThis, lastArgs);
    lastArgs = lastThis = null;
  };

  return function (...args) {
    const now = Date.now();
    if (!lastCall && !leading) lastCall = now;

    const remaining = wait - (now - lastCall);
    lastArgs = args;
    lastThis = this;

    if (remaining <= 0 || remaining > wait) {
      if (timer) {
        clearTimeout(timer);
        timer = null;
      }
      invoke();
    } else if (!timer && trailing) {
      timer = setTimeout(invoke, remaining);
    }
  };
}
```

> 실무에서는 검증된 유틸리티(예: Lodash `debounce`/`throttle`) 사용을 권장합니다.

---

## 3) React 훅 예시

### 3.1 디바운스된 값

```jsx
import { useEffect, useState } from "react";

function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

### 3.2 쓰로틀된 콜백

```jsx
import { useRef, useEffect, useCallback } from "react";

function useThrottle(fn, wait = 200, opts) {
  const fnRef = useRef(fn);
  useEffect(() => {
    fnRef.current = fn;
  }, [fn]);

  return useCallback(
    throttle((...args) => fnRef.current(...args), wait, opts),
    [wait, opts]
  );
}
```

---

## 4) 무한 스크롤에는 무엇을 쓰는가?

- **쓰로틀** 권장
  - 스크롤은 **계속 발생**하고, **빠른 반응**이 중요합니다.
  - 쓰로틀은 **일정 간격마다 즉시/주기적** 반응하여 **지연 체감이 적습니다**.
- **디바운스의 단점**: 사용자가 스크롤을 멈출 때까지 **대기**하므로, 바닥 도달 시점에 **로딩이 늦어질** 수 있습니다.
- **대안**: `IntersectionObserver`로 **Sentinel 요소**가 뷰포트에 들어올 때 로드(브라우저 네이티브, 프레임 친화).

```js
const sentinel = document.querySelector("#sentinel");
const io = new IntersectionObserver(
  (entries) => {
    if (entries.some((e) => e.isIntersecting)) {
      loadMore(); // 다음 페이지 요청
    }
  },
  { root: null, rootMargin: "0px 0px 300px 0px", threshold: 0 }
);

io.observe(sentinel);
```

> 스크롤 이벤트 + 쓰로틀도 가능하지만, **IntersectionObserver가 더 단순하고 배터리/프레임 친화적**입니다.

---

## 5) rAF 기반 스로틀(애니메이션 친화)

- `requestAnimationFrame` 주기에 맞춰 **프레임당 최대 1회** 실행 → 스크롤/드래그 시 **60fps 타겟**에서 자연스러운 업데이트.

```js
function rafThrottle(fn) {
  let ticking = false;
  return function (...args) {
    if (!ticking) {
      ticking = true;
      requestAnimationFrame(() => {
        fn.apply(this, args);
        ticking = false;
      });
    }
  };
}
```

---

## 6) 선택 기준과 튜닝

| 상황                                                      | 권장                                        |
| --------------------------------------------------------- | ------------------------------------------- |
| 실시간 입력 후 **정지 시** 1회 실행(검색 제안, 자동 저장) | **디바운스**(trailing 중심, 200~500ms)      |
| 스크롤/리사이즈 등 **지속 신호**에 **주기적** 반응        | **쓰로틀**(leading+trailing 조합, 50~200ms) |
| 프레임 동기화가 중요한 시각 업데이트                      | **rAF 쓰로틀**                              |
| 뷰포트 진입 시점 트리거                                   | **IntersectionObserver**                    |

- **leading vs trailing**
  - **leading**: 이벤트 시작 즉시 반응(체감 빠름)
  - **trailing**: 마지막 변경을 놓치지 않음(정확성)
  - 필요 시 둘 다 활성화하여 첫 반응 + 마지막 보정

---

## 7) 흔한 함정과 모범 사례

- **컨텍스트/this 손실**: 바인딩이 필요한 메서드는 `fn.bind(...)` 또는 화살표 함수 사용.
- **최신 상태 참조 문제(React)**: 콜백을 **ref에 저장**하거나 `useCallback`으로 의존성 관리.
- **취소/플러시 제어**: 입력 확정 시 **대기 중 디바운스 즉시 실행(flush)** 또는 **취소(cancel)** API가 있으면 활용.
- **지나친 대기 시간**: UX 저하. 측정 기반으로 **적절한 간격** 튜닝.
- **스크롤 핸들러 내부 강제 동기 레이아웃 측정**: `getBoundingClientRect`/스타일 쓰기 혼동 → **레이아웃 스래싱** 주의(읽기와 쓰기 분리).

---

## 8) 체크리스트

- [ ] 입력형 이벤트(검색/자동저장)에 **디바운스**를 적용했는가
- [ ] 스크롤/드래그 등 지속 이벤트에 **쓰로틀/rAF**를 적용했는가
- [ ] React에서 **최신 콜백 참조**를 안전하게 유지했는가
- [ ] 필요 시 **IntersectionObserver**로 단순화했는가
- [ ] leading/trailing 설정을 UX에 맞춰 **정확히 선택**했는가
- [ ] 취소/플러시/클린업을 구현했는가

---

## 9) 요약

- **디바운스**: 연속 입력이 **멈춘 뒤** 1회 실행 → 입력형 작업에 적합.
- **쓰로틀**: **일정 간격마다** 최대 1회 실행 → 스크롤/드래그/무한 스크롤에 적합.
- React에서는 **ref 보관/의존성 관리**와 함께, 상황에 따라 **rAF**나 **IntersectionObserver** 대안을 검토하십시오.
