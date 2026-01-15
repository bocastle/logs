# 프론트엔드 E2E(End‑to‑End) 테스트 한눈에 정리

> 정의: 실제 브라우저 환경에서 **사용자 관점의 핵심 흐름을 처음부터 끝까지** 시뮬레이션하여 검증하는 테스트입니다. Cypress, Playwright 등이 대표 도구입니다.

---

## 1) 왜 필요한가

- **사용자 영향도 높은 오류**를 조기에 발견합니다(로그인·결제·게시/수정 등).
- UI 상호작용, 라우팅, API 호출, 세션·스토리지 등 **전 구성요소의 통합 동작**을 확인합니다.
- 배포 전 **회귀 위험**을 낮추고, 제품 **신뢰성**을 높입니다.

> 유닛 테스트는 “부분 동작” 보증, E2E는 “사용자 여정” 보증. 서로 **보완적**입니다.

---

## 2) 무엇을 테스트할까

- **스모크**: 앱 로딩, 주요 페이지 진입, 기본 탐색
- **핵심 플로우**: 회원가입/로그인, 장바구니/결제, 글 작성/수정/삭제
- **권한/접근 제어**: 로그인 필요 페이지 리디렉션, 역할별 UI/액션
- **오류 처리**: 네트워크 실패·타임아웃 시 사용자 피드백

---

## 3) 도구

- **Cypress**: 직관적 러너, 타임트래블 디버깅, 강력한 개발자 경험
- **Playwright**: 멀티 브라우저(Chromium/Firefox/WebKit), 병렬·샤딩 내장, 네트워크 제어에 강함

---

## 4) 안정적 테스트 설계 원칙

1. **안정 선택자**: `data-testid`/`data-e2e` 사용, CSS/텍스트 의존 최소화
2. **조건 기반 대기**: 고정 `sleep` 대신 요소/응답 조건 대기
3. **테스트 격리**: 케이스별 독립 데이터·클린업 또는 고정 픽스처
4. **페이지 오브젝트**: 셀렉터·행동 캡슐화로 유지보수성 확보
5. **네트워크 전략**: 실서버 스모크 + 실패/엣지 케이스는 모킹
6. **속도/안정화**: 병렬화·샤딩·리트라이·트레이스/비디오 수집

---

## 5) 예시

### 5.1 Playwright

```ts
import { test, expect } from "@playwright/test";

test("로그인 후 프로필 수정", async ({ page }) => {
  await page.goto("https://app.example.com/login");
  await page.getByTestId("email").fill("user@example.com");
  await page.getByTestId("password").fill("pass1234");
  await page.getByTestId("submit").click();
  await page.waitForURL("**/dashboard");

  await page.getByRole("link", { name: "Profile" }).click();
  await page.getByTestId("profile-name").fill("새 이름");
  const [resp] = await Promise.all([
    page.waitForResponse((r) => r.url().endsWith("/api/profile") && r.ok()),
    page.getByTestId("save-profile").click(),
  ]);
  expect(await resp.json()).toMatchObject({ name: "새 이름" });
});
```

### 5.2 Cypress

```js
describe("장바구니 플로우", () => {
  it("상품 담기 및 합계 표시", () => {
    cy.intercept("POST", "/api/cart", { fixture: "cart/add.json" }).as("add");
    cy.visit("/products/42");
    cy.get('[data-e2e="add-to-cart"]').click();
    cy.wait("@add").its("response.statusCode").should("eq", 200);
    cy.get('[data-e2e="cart-count"]').should("contain", "1");
  });
});
```

---

## 6) CI 파이프라인 운영 팁

- **스테이지드 실행**: PR에는 스모크, 메인/릴리즈에는 회귀 세트
- **브라우저 매트릭스**: 최소 지원 브라우저 조합에서 병렬 실행
- **아티팩트**: 실패 시 스크린샷·비디오·네트워크/콘솔 로그 저장
- **환경 고정**: 시드 데이터/테스트 계정·토큰·엔드포인트 외부화

---

## 7) 체크리스트

- [ ] 핵심 사용자 흐름을 선정했는가
- [ ] `data-*` 기반 안정 셀렉터를 사용했는가
- [ ] 고정 `sleep` 대신 조건 대기를 적용했는가
- [ ] 테스트 간 데이터가 격리되는가
- [ ] 모킹과 실서버 사용 기준이 명확한가
- [ ] 병렬·샤딩·리트라이가 설정되어 있는가
- [ ] 실패 아티팩트를 수집·검색할 수 있는가

---

## 8) 요약

- E2E는 **사용자 여정의 신뢰성**을 보증합니다.
- 유닛/통합 테스트와 **함께** 운용할 때 비용 대비 효과가 큽니다.
- **선택자·대기·격리·네트워크 전략**을 표준화하면 플레이키를 통제하고 유지보수를 단순화할 수 있습니다.
