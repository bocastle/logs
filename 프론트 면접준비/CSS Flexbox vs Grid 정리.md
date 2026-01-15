# CSS Flexbox vs Grid 정리

> 두 레이아웃 시스템은 **역할이 다릅니다**. Flexbox는 **1차원(행 또는 열)** 배치에 강하고, Grid는 **2차원(행+열)** 배치에 강합니다. 실제 현장에서는 **함께** 사용하여 최적의 레이아웃을 구성합니다.

---

## 1) 한눈에 비교

| 구분           | Flexbox                                           | Grid                                                  |
| -------------- | ------------------------------------------------- | ----------------------------------------------------- |
| 레이아웃 차원  | 1차원(가로 **또는** 세로)                         | 2차원(가로 **그리고** 세로)                           |
| 사용 초점      | **콘텐츠 중심**(내용 크기/순서에 유연)            | **레이아웃 중심**(격자 정의 후 배치)                  |
| 주 사용처      | 버튼 바, 내비게이션, 태그 리스트, 폼 필드 정렬    | 페이지 프레임, 대시보드, 카드/갤러리, 잡지형 레이아웃 |
| 배치 방식      | 주축(main)/교차축(cross)에 정렬                   | 명시적 행/열 정의 + 자동 배치                         |
| 갭 처리        | `gap` 지원(현대 브라우저)                         | `gap` 기본                                            |
| 정렬           | `justify-content`, `align-items`, `align-content` | 그리드 영역 기준 정렬 + `place-*` 계열                |
| 자동 크기      | 내용 기반 수축/확장(`flex:`, `flex-basis`)        | `fr`, `minmax()`, `auto-fit/fill` 로 격자 크기 제어   |
| 서브 레이아웃  | 중첩 플렉스 컨테이너                              | **`subgrid`**(지원 브라우저), 중첩 그리드             |
| 소스 순서 영향 | **시각 순서=DOM 순서**(주로)                      | 영역 지정으로 **시각 순서 분리 가능**(접근성 주의)    |

---

## 2) 언제 무엇을 선택할까

- **한 줄 또는 한 방향으로 흐르는 콘텐츠**를 정렬해야 하면 **Flexbox**  
  예: 정렬 가능한 버튼 그룹, 태그/칩 리스트, 수평/수직 센터링, 툴바
- **페이지 뼈대/대칭 레이아웃/카드 그리드**를 정의해야 하면 **Grid**  
  예: 헤더–사이드바–메인–푸터, 반응형 카드 갤러리, 대시보드 위젯

> 실무 권장: **바깥 프레임 = Grid**, **컴포넌트 내부 정렬 = Flexbox**.

---

## 3) 대표 문법 요약

### Flexbox

```css
.container {
  display: flex; /* 또는 inline-flex */
  flex-direction: row; /* row | column | row-reverse | column-reverse */
  flex-wrap: wrap; /* nowrap | wrap | wrap-reverse */
  gap: 8px;
  justify-content: space-between; /* 주축 정렬 */
  align-items: center; /* 교차축 정렬 */
}
.item {
  flex: 1 1 200px;
} /* grow shrink basis */
```

### Grid

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  grid-template-rows: masonry; /* 일부 브라우저/실험. 일반적으론 명시적 행 정의 */
  gap: 12px;
}
.item:nth-child(1) {
  grid-column: 1 / span 2; /* 열 병합 */
  grid-row: 1 / span 2; /* 행 병합 */
}
```

> `repeat()`, `fr`, `minmax()`, `auto-fit`/`auto-fill`은 **반응형 카드 레이아웃**에서 매우 유용합니다.

---

## 4) 반응형 설계 패턴

### 4.1 카드 갤러리(Grid)

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
}
```

### 4.2 버튼 바(Flexbox)

```css
.toolbar {
  display: flex;
  gap: 8px;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
}
```

### 4.3 두 단 레이아웃(Grid + Flex 혼합)

```css
.page {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto; /* 헤더, 메인, 푸터 */
  min-height: 100vh;
}
.header {
  grid-column: 1 / -1;
}
.sidebar {
  grid-column: 1;
}
.main {
  grid-column: 2;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.footer {
  grid-column: 1 / -1;
}
```

---

## 5) 정렬과 간격

- Flexbox: 주축 **분배**(`justify-*`) + 교차축 **정렬**(`align-*`), 여러 줄일 때 `align-content`/`place-content`.
- Grid: 셀/트랙 기준 정렬(`justify-items`, `align-items`), 컨테이너 전체 정렬(`justify-content`, `align-content`).  
  요소 단위 오버라이드: `justify-self`, `align-self`, `place-self`.

> 두 시스템 모두 `gap` 사용을 권장(마진 대신). 방향/정렬 변화에도 안정적으로 간격이 유지됩니다.

---

## 6) 자동 배치와 소스 순서

- Grid의 **영역 배치**는 시각 순서를 DOM과 분리할 수 있으나, **키보드 포커스 순서/스크린 리더** 접근성에 유의해야 합니다.
- Flexbox는 DOM 순서와 시각 순서가 대체로 일치합니다. `order`로 변경 가능하지만 **접근성 문제**를 유발할 수 있습니다.

---

## 7) 성능과 유지보수

- 단순 정렬에는 **Flexbox가 가볍고 직관적**입니다.
- 복잡한 페이지 프레임/교차 정렬/셀 병합이 필요하면 **Grid가 규칙적으로 관리**됩니다.
- 클래스 네이밍과 격자 정의를 컴포넌트/디자인 토큰과 **일관**시키면 유지보수가 쉬워집니다.

---

## 8) Subgrid와 고급 기능(Grid)

- **`subgrid`**: 자식 그리드가 부모의 **행/열 트랙 정의를 상속**(일관 정렬에 유리).  
  지원 현황을 확인하고, 미지원 환경에서는 **중첩 그리드 + CSS 변수**로 유사 구현을 고려합니다.
- **`grid-auto-flow`**: `row`/`column`/`dense` 자동 배치 규칙.
- **명시/암시 트랙**: `grid-auto-rows`/`grid-auto-columns`로 암시적 트랙 크기 제어.

---

## 9) 흔한 실수와 대안

- **마진으로 간격 관리** → `gap`으로 통일(방향 전환 시 유지).
- **Flex로 2차원 격자 흉내** → 정렬·줄바꿈·높이 맞춤 문제가 누적. **Grid**가 더 단순.
- **Grid로 작은 수평 정렬** → 과설계. 버튼 줄·폼 정렬은 **Flex**가 간편.
- **`order`/영역 배치로 시각 순서만 바꿈** → **접근성** 점검(포커스 순서/리더 순서).

---

## 10) 선택 가이드 체크리스트

- [ ] 한 방향 정렬인가(→ Flex) 아니면 행+열 동시 제어인가(→ Grid)?
- [ ] 간격/정렬을 `gap`/`place-*`로 일관 제어하는가?
- [ ] 반응형에서 열 수 자동 조정이 필요한가(`fr`/`minmax`/`auto-fit`)?
- [ ] 접근성(포커스/리딩 순서)이 DOM 순서와 일치하는가?
- [ ] 유지보수 관점에서 규칙(그리드 템플릿/토큰)이 명확한가?

---

## 11) 요약

- **Flexbox = 1차원 콘텐츠 정렬**, **Grid = 2차원 레이아웃 설계**.
- **페이지 프레임은 Grid**, **컴포넌트 내부 정렬은 Flex**가 기본 전략입니다.
- `gap`, `fr`, `minmax()`, `auto-fit/fill`, `place-*` 등 핵심 속성을 숙지하면 대부분의 레이아웃 문제를 간결하게 해결할 수 있습니다.
