# DOM 이벤트 전파(Event Propagation) 정리

> 이벤트가 발생했을 때 **어떤 순서**로 **어떤 노드**에서 핸들러가 호출되는지에 대한 규칙입니다.  
> 핵심: **캡처링 → 타깃 → 버블링**의 3단계, 그리고 **위임·차단·기본 동작**의 정확한 이해입니다.

---

## 1) 전파 3단계

1. **캡처링(Capturing)**: `window/document` → `html` → … → **타깃 노드**로 **내려감**.
   - `addEventListener(type, handler, { capture: true })` 로 **캡처 단계**에서 처리 가능
2. **타깃(Target)**: 이벤트가 실제로 발생한 **타깃 요소**에서 핸들러 실행
3. **버블링(Bubbling)**: 타깃 → 상위 조상으로 **올라감**(부모 → … → `document/window`)
   - 기본값은 **버블 단계** 리스너(`capture:false`)

> 대부분의 UI 이벤트는 버블링됩니다. 단, 일부 이벤트는 버블링하지 않거나(예: `blur`, `focus`), 별도 이벤트로 대체(`focusin/focusout`은 버블링)됩니다.

---

## 2) 이벤트 객체의 핵심 필드

- `event.target`: **실제 발생 지점**(가장 안쪽 노드)
- `event.currentTarget`: **현재 실행 중인 핸들러가 바인딩된 노드**
- `event.eventPhase`: **1=캡처**, **2=타깃**, **3=버블**
- `event.bubbles`: 해당 이벤트가 **버블링 가능한지**
- `event.cancelable`: **기본 동작 취소**(`preventDefault`) 가능 여부
- `event.defaultPrevented`: 기본 동작 취소 여부
- `event.composedPath()`: **전파 경로(배열)** — **Shadow DOM** 경계 포함/제외 여부는 이벤트 **composed** 특성에 따름

---

## 3) 전파 제어

- `event.stopPropagation()`
  - **현재 노드 기준으로 더 이상 상·하위로 전파하지 않음**(같은 노드의 다른 핸들러는 계속 실행)
- `event.stopImmediatePropagation()`
  - **같은 노드에 등록된 나머지 핸들러도 즉시 중단**
- `event.preventDefault()`
  - 링크 이동, 폼 제출, 텍스트 선택 등 **브라우저 기본 동작만 취소**
  - **전파와는 별개**이므로 필요 시 **둘 다 호출** 가능

> 실무 팁: **컴포넌트 캡슐화**가 필요할 때만 전파 차단을 사용하시고, 가능한 한 **이벤트 위임**으로 해결하는 편이 유지보수에 유리합니다.

---

## 4) 이벤트 리스너 옵션

```js
element.addEventListener("click", handler, {
  capture: false, // 캡처 단계에서 실행할지
  once: true, // 한 번 실행 후 자동 제거
  passive: true, // handler 안에서 preventDefault 호출하지 않을 것임을 힌트(스크롤 최적화)
  signal: abortController.signal, // AbortController로 해제 제어
});
```

- **`passive: true`**: `touchstart`/`wheel`/`scroll` 등에서 **스크롤 성능** 개선
  - passive일 때 `preventDefault()`는 무시되므로 **호출하지 않아야** 합니다.
- **`signal`**: 정리 시 `abortController.abort()`로 **여러 리스너를 일괄 해제**

---

## 5) 이벤트 위임(Event Delegation)

- **원리**: 상위 컨테이너에 **하나의 핸들러**만 두고, `event.target`/`closest()`로 실제 타깃을 판별
- **장점**: 동적 노드 추가/제거에 강함, 메모리/등록 비용 감소, 성능 향상

```js
list.addEventListener("click", (e) => {
  const item = e.target.closest(".todo-item");
  if (!item || !list.contains(item)) return;
  // 아이템 클릭 처리
});
```

> 위임 시 **버블링 가능한 이벤트**인지 확인하십시오(필요 시 `focusin`/`focusout` 사용).

---

## 6) 이벤트 순서와 기본 동작

- **클릭 흐름 예시**: `mousedown` → `mouseup` → `click` (→ `dblclick`)
  - 텍스트 선택/드래그와 같은 **기본 동작**이 중간에 개입할 수 있음
- **폼 제출**: `submit`(버블링) 시 `preventDefault()`로 제출 취소 가능
- **링크**: `click`에서 `preventDefault()`로 **페이지 전환 방지**

---

## 7) 포인터·터치·마우스의 관계(요약)

- 최신 사양은 **Pointer Events** 권장(`pointerdown/move/up/cancel`) — 마우스/펜/터치를 **일원화**
- 터치에서 **수직 스크롤**을 막으려면 passive 리스너를 사용하지 말고, 필요 시 **CSS `touch-action`** 으로 제어

---

## 8) Shadow DOM과 전파

- Shadow 경계에서 전파는 **구성된 경로(composed path)** 규칙을 따름
- 이벤트가 `composed: true`이면 **Shadow 경계를 넘어 상위로 버블** 가능(예: `click`)
- 캡슐화된 컴포넌트에서 외부 노출이 필요하면 **커스텀 이벤트**를 `new CustomEvent(type, { bubbles: true, composed: true })` 로 디스패치

```js
this.dispatchEvent(
  new CustomEvent("card-open", {
    detail: { id },
    bubbles: true,
    composed: true,
  })
);
```

---

## 9) 실무 패턴과 주의 사항

- **전파 차단 남용 금지**: 상위 기능을 우연히 깨뜨릴 수 있음
- **핸들러 최소화**: 위임 + 조건 분기
- **성능**: 스크롤/포인터 이동 등 고빈도 이벤트는 **쓰로틀/rAF** 조합
- **접근성**: 클릭만 처리하지 말고 **키보드 이벤트**(`keydown`/`Enter`/`Space`)도 함께 고려
- **정리(해제)**: SPA 내 페이지 전환 시 누수 방지 — `AbortController`/프레임워크 언마운트 훅 활용

---

## 10) 체크리스트

- [ ] 필요한 단계(캡처/버블)를 올바르게 선택했는가
- [ ] `target` vs `currentTarget`을 혼동하지 않았는가
- [ ] 위임이 가능한가(버블링 이벤트인지 확인)
- [ ] `preventDefault`와 `stopPropagation`의 역할을 구분했는가
- [ ] 고빈도 이벤트에 `passive`/쓰로틀/rAF 등을 적용했는가
- [ ] Shadow DOM 경계와 `composedPath`를 이해하고 필요한 이벤트를 노출했는가
- [ ] 리스너 해제 전략(`once`/`signal`)을 갖추었는가

---

## 11) 요약

- 이벤트 전파는 **캡처링 → 타깃 → 버블링** 순서로 진행됩니다.
- 제어 수단: `capture` 옵션, `stopPropagation`/`stopImmediatePropagation`, `preventDefault`.
- **이벤트 위임**으로 핸들러를 단순화하고 성능·유지보수성을 높일 수 있습니다.
- Shadow DOM/Pointer Events/리스너 옵션까지 이해하면 **복잡한 상호작용**도 안전하게 관리할 수 있습니다.
