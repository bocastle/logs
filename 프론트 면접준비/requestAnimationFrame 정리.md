# requestAnimationFrame 정리

## 1) requestAnimationFrame이란?

`requestAnimationFrame(callback)`(이하 rAF)은 **브라우저의 화면 갱신(리페인트/컴포지팅) 주기**에 맞춰 콜백을 실행하도록 요청하는 API입니다.  
주로 **애니메이션, 스크롤 기반 인터랙션, 드래그 중 위치 업데이트** 같은 “프레임 단위 UI 갱신”에 사용됩니다.

- 브라우저가 “다음 프레임을 그리기 직전” 적절한 시점에 콜백을 호출합니다.
- 콜백은 보통 초당 **60fps(약 16.7ms 간격)** 또는 디스플레이 주사율에 맞춰 호출됩니다(120Hz면 더 자주).

---

## 2) 왜 쓰나요? (setTimeout/setInterval과의 차이)

### (1) 화면 갱신 타이밍에 맞춘 실행

`setTimeout/setInterval`은 “시간” 중심이라 브라우저 렌더링 타이밍과 어긋날 수 있습니다.  
그 결과:

- 프레임 드롭
- 불필요한 중복 렌더링
- 끊기는 애니메이션

rAF는 브라우저가 렌더링하기 좋은 시점에 맞춰 실행되어, 같은 작업이라도 **더 부드럽고 효율적**입니다.

### (2) 주사율에 맞춰 자동 조절

모니터가 60Hz/120Hz/144Hz 등일 때 rAF 호출도 **해당 주사율에 맞게 자연스럽게 동기화**됩니다.

### (3) 백그라운드 탭에서 실행 억제/절전

탭이 백그라운드이거나 화면이 숨겨진 상태일 때는 rAF 호출이 줄어들거나 멈춰서  
**CPU/배터리 사용량을 절약**합니다.

---

## 3) 기본 사용 패턴

### 반복 애니메이션 루프

```js
let rafId;

function animate() {
  // 1) 상태 업데이트
  // 2) DOM/Canvas 반영
  rafId = requestAnimationFrame(animate);
}

rafId = requestAnimationFrame(animate);

// 멈추기
cancelAnimationFrame(rafId);
```

### timestamp 활용(권장)

rAF 콜백은 실행 시점의 고정밀 타임스탬프를 인자로 받습니다.

```js
let rafId;
let start;

function animate(ts) {
  if (start == null) start = ts;
  const elapsed = ts - start;

  // elapsed 기반으로 부드럽게 이동(프레임 간격이 달라도 일정 속도 유지)
  // 예: x = speed * elapsed

  if (elapsed < 1000) {
    rafId = requestAnimationFrame(animate);
  }
}

rafId = requestAnimationFrame(animate);
```

---

## 4) 어디에 쓰면 좋은가?

- CSS transform 기반 애니메이션(특히 JS로 위치/회전/스케일을 갱신할 때)
- Canvas/WebGL 렌더 루프
- 스크롤/드래그 중 위치 갱신(연속 업데이트)
- “프레임당 1회만 반영”이 필요한 작업(리사이즈/스크롤 이벤트 디바운싱 대체 용도)

---

## 5) 이벤트 루프 관점: 태스크 큐/마이크로태스크와 다르다

rAF 콜백은 일반적인 **매크로태스크 큐(setTimeout)** 나 **마이크로태스크 큐(Promise.then)** 와 같은 큐에서만 움직이지 않습니다.

- rAF 콜백은 브라우저가 내부적으로 관리하는 “animation frame callbacks” 목록에 등록되고,
- 브라우저는 **렌더링 단계 직전**(프레임을 그리기 직전)에 등록된 콜백을 실행합니다.
- 실행 순서 관점에서 보면, 대체로 “이벤트 처리 → 스크립트 실행 → (필요시) 레이아웃/페인트 준비 → rAF → 렌더링” 식의 흐름으로 이해하면 실무에 도움이 됩니다.

> 요약: rAF는 “렌더링 파이프라인에 밀접한 스케줄링”을 갖는 별도 메커니즘입니다.

---

## 6) 주의사항

- rAF 안에서 **무거운 작업**을 하면 프레임이 밀려 끊김이 발생합니다(16ms 예산 내 처리 권장).
- DOM 측정(layout 읽기)과 DOM 변경(layout 쓰기)을 한 프레임에서 섞으면 레이아웃 스래싱이 발생할 수 있으니 주의합니다.
- 애니메이션은 가능하면 CSS(특히 transform/opacity)로 처리하는 편이 단순하고 최적화가 잘 되는 경우가 많고, JS 애니메이션이 필요할 때 rAF를 선택하는 것이 좋습니다.

---

## 한 줄 요약

`requestAnimationFrame`은 **브라우저의 프레임 렌더링 타이밍에 맞춰 콜백을 실행**해 애니메이션을 부드럽게 만들고, 불필요한 렌더링과 리소스 낭비를 줄이는 API입니다.
