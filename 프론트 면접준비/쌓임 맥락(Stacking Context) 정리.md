# 쌓임 맥락(Stacking Context) 정리

## 1) 쌓임 맥락이란?

쌓임 맥락(Stacking Context)은 브라우저가 요소들을 **z축(앞/뒤)** 방향으로 어떻게 겹쳐서 그릴지 결정하기 위해 만드는 **독립적인 레이어(가상 3D 공간)** 입니다.  
핵심은 다음 한 줄입니다.

- **z-index는 “같은 쌓임 맥락 내부”에서만 비교됩니다.**  
  어떤 요소의 z-index가 매우 커도, **다른 쌓임 맥락에 속해 있으면** 기대와 다르게 뒤에 깔릴 수 있습니다.

---

## 2) 기본 쌓임 순서(쌓임 맥락이 특별히 없을 때)

같은 쌓임 맥락(보통 루트) 안에서 겹침 순서는 대체로 다음 요소들의 조합으로 결정됩니다.

- **DOM 순서**: 나중에 나온 요소가 더 위에 그려지기 쉽습니다(겹칠 때).
- **position + z-index**: position이 있고 z-index가 의미 있게 적용되는 조건이면, z-index가 클수록 앞쪽에 그려집니다.

실무에서는 아래 3가지를 먼저 확인하면 대부분 해결됩니다.

1. **쌓임 맥락을 먼저 파악한다.**
2. **같은 맥락 안에서만 z-index를 비교한다.**
3. **DOM 순서도 결과에 영향을 준다.**

---

## 3) 새로운 쌓임 맥락이 생성되는 대표 조건

다음과 같은 경우, 요소가 **새 쌓임 맥락을 생성**합니다(대표 사례).

### (1) position + z-index

- `position: relative | absolute` 이면서 `z-index: auto`가 **아닌 경우**
- `position: fixed | sticky` 인 경우(대부분 새 맥락을 형성)

### (2) 레이아웃 컨테이너에서의 z-index

- `display: flex | grid` 인 요소가 `z-index`를 갖고 특정 조건을 만족하는 경우

### (3) 시각 효과(합성 레이어) 관련 속성

- `opacity < 1`
- `transform` (예: translate, scale 등)
- `filter`, `backdrop-filter`
- `perspective`, `mix-blend-mode`, `isolation: isolate` 등

> 팁: “왜 z-index가 안 먹지?”의 흔한 원인은 **transform/opacity로 쌓임 맥락이 생긴 경우**입니다.

---

## 4) 왜 z-index가 큰데도 위로 안 올라가나요?

**서로 다른 쌓임 맥락이면** z-index는 비교 대상이 아닙니다.

- A 요소가 쌓임 맥락을 만들고 그 안에 A-1(자식)이 `z-index: 9999`라도,  
  **A 쌓임 맥락 자체가** B 쌓임 맥락보다 아래면 A-1은 B보다 위로 못 올라갑니다.

---

## 5) 예시로 이해하기

```html
<div style="position: relative; z-index: 1;">
  A 요소 (z-index: 1)
  <div style="position: absolute; z-index: 999;">A-1 요소 (z-index: 999)</div>
</div>

<div style="position: relative; z-index: 2;">B 요소 (z-index: 2)</div>
```

### 해석 흐름

- A는 `position: relative` + `z-index: 1` → **쌓임 맥락 생성**
- B도 `position: relative` + `z-index: 2` → **쌓임 맥락 생성**
- 최상위(루트) 쌓임 맥락에서 A(1) vs B(2) 비교 → **B가 더 위**
- A-1의 `z-index: 999`는 **A의 쌓임 맥락 내부에서만** 의미  
  → 전체적으로 **B > A-1 > A** 순서

---

## 6) 실무에서 자주 겪는 함정

### (1) 부모에 `transform`이 들어가면 fixed 기준이 꼬일 수 있음

부모에 `transform`이 있으면 자식의 `position: fixed`가 기대와 다르게 동작하거나,
겹침 우선순위가 꼬일 수 있습니다.

### (2) 모달이 header 뒤로 숨어버림

모달이 어떤 컨테이너 내부에 렌더링되고,
그 컨테이너가 쌓임 맥락을 만들면(예: `transform: translateZ(0)`),
모달 z-index를 높여도 밖의 header보다 뒤에 깔릴 수 있습니다.

---

## 7) 해결 전략(현업 기준)

1. **원인 파악:** DevTools에서 부모 체인을 따라가며  
   `transform/opacity/position+z-index` 같은 “쌓임 맥락 생성자”를 찾습니다.
2. **같은 쌓임 맥락으로 올리기:** 모달/드롭다운은 보통 `body` 직하로 포탈을 쓰거나,  
   최상위 레이어 컨테이너를 둡니다.
3. **z-index 토큰화:** 숫자 난사 대신  
   `--z-modal: 1000` 같은 토큰으로 계층을 고정합니다.

---

## 한 줄 요약

- **z-index는 절대값이 아니라 “같은 쌓임 맥락 안에서의 상대 비교”입니다.**
