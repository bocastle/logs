# 프론트엔드 E2E(End‑to‑End) 테스트 정리

> 정의: 실제 브라우저 환경에서 **사용자 관점의 전체 흐름**(페이지 이동·입력·클릭·API 연동 등)을 **처음부터 끝까지 시뮬레이션**하여 검증하는 테스트입니다. 대표 도구로 **Cypress, Playwright**가 있습니다.

---

## 1) 왜 필요한가

- **사용자에게 직접 영향**을 주는 오류를 조기에 발견하여 배포 리스크를 낮춥니다.
- **UI 상호작용 + 라우팅 + 세션/스토리지 + API 호출**이 결합된 **통합 동작**을 확인합니다.
- 중요한 사용자 흐름(로그인, 결제, 게시/수정 등)에 대해 **제품 신뢰성**을 보장합니다.

> 유닛 테스트는 “부분 동작”을 검증하고, E2E는 “사용자 여정”을 검증하므로 **상호 보완**이 필요합니다.

---

## 2) 무엇을 테스트할지(우선순위)

- **스모크 플로우**: 앱 로딩, 핵심 페이지 진입, 기본 내비게이션
- **핵심 비즈니스 플로우**: 회원가입/로그인, 장바구니/결제, 글 작성·수정·삭제
- **권한/접근 제어**: 인증 필요 페이지의 리디렉션, 역할에 따른 UI/기능
- **에러 처리**: 네트워크 오류/타임아웃 시 사용자 피드백

---

## 3) 도구 개요

- **Cypress**: 직관적 러너와 타임 트래블 디버깅, 개발 경험이 뛰어남
- **Playwright**: Chromium/Firefox/WebKit **멀티 브라우저**, 병렬·샤딩·네트워크 제어에 강함

---

## 4) 안정적 E2E 설계 원칙

1. **안정 선택자 사용**: `data-testid`·`data-e2e` 등 의미 기반 셀렉터
2. **조건 기반 대기**: `sleep` 지양, 요소·응답 조건으로 대기
3. **테스트 격리**: 케이스 간 데이터 독립(시드·클린업·고유 키)
4. **페이지 오브젝트 패턴**: 셀렉터·액션 캡슐화로 유지보수 비용 절감
5. **네트워크 전략**: 스모크는 실서버, 엣지/실패 케이스는 모킹 병행
6. **속도·안정화**: 병렬/샤딩/리트라이, 실패 시 스크린샷·비디오/로그 수집

---

## 5) 간단 예시

### Playwright

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

### Cypress

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

## 6) 요약

- E2E 테스트는 **사용자 시나리오의 신뢰성**을 보장합니다.
- **유닛/통합 테스트와 병행**할 때 비용 대비 효과가 극대화됩니다.
- **안정 셀렉터·조건 대기·데이터 격리·네트워크 전략**을 표준화하여 **플레이키**를 줄이십시오.
