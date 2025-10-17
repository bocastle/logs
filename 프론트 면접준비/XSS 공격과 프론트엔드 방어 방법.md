# XSS 공격과 프론트엔드 방어 방법

XSS(Cross-Site Scripting)는 공격자가 신뢰할 수 있는 웹사이트에 악성 스크립트를 삽입하여 사용자 브라우저에서 실행되게 하는 공격입니다. 이를 통해 쿠키 탈취, 세션 하이재킹, 피싱 등이 가능합니다.

XSS 공격은 크게 세 가지 유형이 있습니다.

첫번째로 저장형(Stored) XSS입니다. 악성 스크립트가 서버에 저장되어 다른 사용자가 해당 페이지를 방문할 때 실행됩니다.

두번째는 반사형(Reflected) XSS입니다. URL 파라미터 등을 통해 전달된 악성 스크립트가 서버 응답에 포함되어 실행됩니다.

마지막으로 DOM 기반 XSS입니다. 클라이언트 측 스크립트가 DOM을 동적으로 조작할 때 발생합니다.

해결 방법
이러한 보안문제를 해결하기 위한 방법으로는 입력 검증과 출력 이스케이핑이 있습니다. 사용자 입력을 적절히 검증하고, HTML 출력 시 특수 문자를 이스케이프 처리합니다.

아래처럼 element.innerHTML 대신 element.textContent를 이용하거나, DOMPurify를 사용하여 사용자의 입력값에 대해서 특수 문자와 입력 검증을 해줄 수 있습니다.

```js
// ❌ 잘못된 방법
element.innerHTML = userInput;

// ✅ 올바른 방법
element.textContent = userInput; // 자동 이스케이프

// ✅ HTML이 필요한 경우 DOMPurify 사용
import DOMPurify from "dompurify";

element.innerHTML = DOMPurify.sanitize(userInput);
```

혹은, 메타 태그에 Content-Security-Policy(CSP)를 설정하여 브라우저에 실행을 허용할 컨텐츠 소스를 제한할 수 있습니다.

```html
<!-- HTTP 헤더 또는 메타 태그로 설정 -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self' https://trusted-cdn.com;"
/>
```

마지막으로는 HttpOnly 쿠키입니다. 쿠키에 HttpOnly 플래그를 설정하여 자바스크립트를 통한 접근을 차단할 수 있습니다.

---

## 문제

- XSS(Cross-Site Scripting)는 **신뢰된 웹사이트에 악성 스크립트가 삽입되어 사용자 브라우저에서 실행**되는 공격입니다.
- 결과적으로 **쿠키 탈취, 세션 하이재킹, 피싱, UI 변조** 등 심각한 피해가 발생할 수 있습니다.

## 특징

- **저장형(Stored)**: 악성 스크립트가 서버(예: DB)에 저장되어 다른 사용자에게 전파됩니다.
- **반사형(Reflected)**: 요청(예: URL 파라미터)이 응답에 반영되어 즉시 실행됩니다.
- **DOM 기반**: 클라이언트 측 JS가 DOM을 직접 조작할 때 발생합니다 (서버 응답과 무관).

## 장점 (공격자 관점)

- 서버 취약점 없이도 **프론트엔드 취약점만으로 공격 가능**.
- 피해가 브라우저에서 직접 발생하므로 **사용자 영향 즉시 발생**.

## 단점 (공격자 관점)

- CSP, 브라우저 보안, HttpOnly 쿠키 등으로 **차단 가능성 높음**.
- 탐지되면 법적/운영상 제재 대상이 될 수 있음.

## 최적화 (프론트엔드에서의 방어 방법)

1. **출력 이스케이핑 (우선 적용)**

   - 텍스트 삽입 시 `textContent` 사용: 자동 이스케이프.

   ```js
   // ❌ 잘못된 방법
   element.innerHTML = userInput;

   // ✅ 올바른 방법
   element.textContent = userInput; // 자동 이스케이프
   ```

2. **신뢰된 HTML만 허용해야 할 경우: DOMPurify 사용**

   ```js
   import DOMPurify from "dompurify";

   // 사용자의 HTML을 허용해야 한다면 sanitize 후 삽입
   element.innerHTML = DOMPurify.sanitize(userInput);
   ```

3. **입력 검증 (서버 + 클라이언트)**

   - 화이트리스트(허용된 포맷) 적용, 길이·문자 타입 제한, 특수문자 필터링.

4. **Content-Security-Policy(CSP) 적용**

   - 스크립트/스타일/이미지 출처를 제한하여 인라인 스크립트/`eval` 사용을 방지.

   ```html
   <!-- HTTP 헤더 또는 메타 태그로 설정 -->
   <meta
     http-equiv="Content-Security-Policy"
     content="default-src 'self'; script-src 'self' https://trusted-cdn.com;"
   />
   ```

5. **HttpOnly 및 Secure 쿠키**

   - `HttpOnly`로 JS에서 쿠키 접근 차단 → XSS로 인한 쿠키 탈취 방지
   - `Secure`로 HTTPS에서만 전송

6. **프레임워크 기본 보호 기능 활용**

   - React, Vue 등은 기본적으로 출력 이스케이프를 제공(단, `dangerouslySetInnerHTML`, `v-html` 등은 주의).

7. **정기적 보안 점검**
   - SAST/DAST, 자동 스캐너, 펜테스트로 취약점 발견 및 수정.

## 결론

- 본문(원문)은 요청하신 대로 **그대로 유지**했습니다.
- 요약하면 **입력 검증 + 출력 이스케이핑 → 필요 시 DOMPurify → CSP 적용 → HttpOnly/ Secure 쿠키**를 조합해 다층 방어를 구축하면 프론트엔드 관점에서 XSS 위험을 효과적으로 줄일 수 있습니다.

---

### 코드/예시(Obsidian에서 붙여넣기 좋음)

```js
// 예시: 안전한 텍스트 삽입
const el = document.getElementById("out");
el.textContent = userInput; // 안전

// 예시: 신뢰된 HTML 허용 시
import DOMPurify from "dompurify";
el.innerHTML = DOMPurify.sanitize(userInput);
```

```html
<!-- 예시: CSP 메타 태그 -->
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self' https://trusted-cdn.com;"
/>
```
