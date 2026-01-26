# event.target vs event.currentTarget 정리

## 1) 한 줄 요약

- **`event.target`**: _이벤트가 실제로 발생한 요소_ (가장 안쪽, 사용자가 직접 상호작용한 노드)
- **`event.currentTarget`**: _현재 이벤트 리스너가 바인딩된 요소_ (리스너를 붙인 노드)

> 이벤트 버블링 중에는 **target은 동일**하게 유지되고, **currentTarget은 단계마다 바뀝니다.**

---

## 2) 이벤트 전파와의 관계

브라우저는 이벤트를 **캡처링 → 타깃 → 버블링** 순으로 전파합니다.

- **캡처링/버블링 어디서든** `target`은 변하지 않고 최초 발생 요소를 가리킵니다.
- 반면 **리스너가 붙은 요소마다** `currentTarget`은 해당 요소로 바뀝니다.

```html
<div id="parent">
  <button id="child">Click</button>
</div>
<script>
  document.getElementById("parent").addEventListener("click", (e) => {
    console.log("target:", e.target.id); // 'child'
    console.log("currentTarget:", e.currentTarget.id); // 'parent'
  });
</script>
```

---

## 3) 실전 예시 (이벤트 위임)

리스트 항목마다 리스너를 붙이지 않고 **부모 하나에만** 붙이는 패턴입니다.

```html
<ul id="menu">
  <li data-action="open">Open</li>
  <li data-action="save">Save</li>
  <li data-action="close">Close</li>
</ul>

<script>
  const menu = document.getElementById("menu");
  menu.addEventListener("click", (e) => {
    // 항상 발생 지점(자식)을 기준으로 분기
    const li = e.target.closest("li");
    if (!li || !menu.contains(li)) return; // 안전 가드
    switch (li.dataset.action) {
      case "open":
        /* ... */ break;
      case "save":
        /* ... */ break;
      case "close":
        /* ... */ break;
    }
    // 필요 시 currentTarget(=menu) 활용 가능
    // e.currentTarget === menu
  });
</script>
```

**포인트**

- `e.target`을 바로 신뢰하지 말고 `closest`로 의도한 셀렉터를 재확인합니다.
- `e.currentTarget`은 위임의 “리스너 보유자(부모)”를 가리킵니다.

---

## 4) 자주 발생하는 실수와 예방책

1. **스타일/아이콘 클릭 시 target이 예상과 다름**
   - 해결: `e.target.closest('.btn')` 등으로 상위 의도 요소를 찾습니다.
2. **버블링을 의도치 않게 허용**
   - 해결: 특정 조건에서만 `e.stopPropagation()`을 사용하되 남용은 지양합니다.
3. **텍스트 노드가 target이 되는 경우**
   - 해결: `nodeType` 확인 또는 `closest` 사용으로 엘리먼트 기준 처리합니다.
4. **캡처링 리스너에서 혼동**
   - 캡처링 단계에서도 `target`은 변하지 않습니다. `currentTarget`만 해당 단계의 리스너 소유자를 가리킵니다.

---

## 5) React에서의 주의점

- React의 SyntheticEvent도 개념은 동일합니다.  
  `event.target`은 실제 클릭된 하위 요소, `event.currentTarget`은 **핸들러가 바인딩된 컴포넌트의 DOM 노드**입니다.
- 타입가드가 필요할 수 있습니다. (예: TypeScript에서 `e.target as HTMLButtonElement`)
- 이벤트가 풀링되던 과거 버전과 달리 최신 React는 기본적으로 풀링하지 않지만, 비동기 사용 시엔 **이벤트 참조 유효성**을 유의하세요.

---

## 6) 속성 비교표

| 구분                  | 의미                  | 값이 가리키는 대상                      | 전파 중 변화        | 대표 사용처                   |
| --------------------- | --------------------- | --------------------------------------- | ------------------- | ----------------------------- |
| `event.target`        | 실제 이벤트 발생 지점 | 가장 안쪽의 노드(텍스트 노드 포함 가능) | **변하지 않음**     | 위임 시 분기, 클릭 원소 판별  |
| `event.currentTarget` | 리스너가 연결된 요소  | 현재 처리 중인 리스너 소유 엘리먼트     | **리스너마다 바뀜** | 컨테이너 기준 로직, 범위 체크 |

---

## 7) 스니펫: 안전한 버튼 처리기

```ts
// TS 예시
container.addEventListener("click", (e) => {
  const btn = (e.target as HTMLElement).closest("button");
  if (!btn || !(e.currentTarget as HTMLElement).contains(btn)) return;
  // 여기서부터 btn은 의도한 버튼
});
```

---

## 8) 체크리스트

- 위임 시 **`closest` + `contains`**로 의도한 엘리먼트만 처리했는가
- **버블링/캡처링 위치**를 명확히 이해하고 리스너 옵션을 설정했는가 (`capture: true` 등)
- React/TS 환경에서는 **타입/이벤트 유효성**을 고려했는가
