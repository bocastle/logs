# localStorage vs sessionStorage 정리

> 요약: 두 API 모두 **키-값**(문자열 기반) 저장을 제공하지만, **지속성·가시 범위·세션 생명주기**가 다릅니다. 민감 정보는 저장하지 말고 **HttpOnly 쿠키**나 **서버 세션/토큰 보관 전략**을 사용하십시오.

---

## 1) 공통점

- 브라우저의 **Web Storage API**: `setItem(key, value)`, `getItem(key)`, `removeItem(key)`, `clear()`, `key(index)`
- **문자열만 저장**(객체는 `JSON.stringify/parse` 필요)
- **동기(synchronous) API**이므로 대량 호출은 메인 스레드를 블로킹할 수 있음
- **동일 출처(Same-Origin)**에만 접근 가능
- 변경 시 **`storage` 이벤트** 발생(단, 변경이 일어난 탭 자신은 이벤트 미수신)

---

## 2) 주요 차이점

| 구분      | localStorage                                        | sessionStorage                                          |
| --------- | --------------------------------------------------- | ------------------------------------------------------- |
| 수명      | 브라우저/OS 재시작 후에도 **영속**                  | **탭·창 세션 동안만** 유지(닫히면 삭제)                 |
| 공유 범위 | **동일 출처의 모든 탭/창**에서 공유                 | **탭(창) 단위로 분리**, 새 탭 복제 시 복제본만 공유     |
| 용도      | 장기 설정(테마), 최근 본 항목, 비로그인 장바구니 등 | 일시 상태(페이지 위저드 진행도), 탭 한정 임시 데이터 등 |

---

## 3) 사용 예시

```ts
// localStorage: 다크 모드 설정
const key = "theme";
const theme = localStorage.getItem(key) ?? "light";
document.documentElement.dataset.theme = theme;

// sessionStorage: 탭 한정 임시 필터
sessionStorage.setItem(
  "filter",
  JSON.stringify({ q: "react", sort: "recent" })
);
const filter = JSON.parse(sessionStorage.getItem("filter") ?? "{}");
```

---

## 4) 한계와 문제점

1. **보안**

- JS로 접근 가능 → **XSS에 노출**되면 저장 값 탈취 위험.
- **HttpOnly 쿠키가 아님**(스크립트 접근 차단 불가).
- **민감 정보(액세스 토큰·개인정보) 저장 금지** 권장.

2. **용량/호환성**

- 브라우저·플랫폼별 용량 상이(대략 수 MB 수준). **저용량 저장소**.
- 프라이빗 모드(특히 iOS Safari)에서 **용량 제한/세션 격리** 차이 존재.

3. **동기 API 성능**

- 대량 데이터 read/write는 **메인 스레드 블로킹**.
- 대안: **IndexedDB(비동기, 대용량, 구조화 데이터)** 검토.

4. **문자열 직렬화**

- 객체·날짜·Map 등은 직접 **직렬화/역직렬화** 필요, 버전 호환 로직 필요.

---

## 5) 보안 모범 사례

- **민감 정보 저장 금지**: 액세스/리프레시 토큰, 사용자 PII는 Web Storage에 두지 않음.
- 인증은 **HttpOnly + Secure 쿠키** 기반 세션/토큰 전략을 고려. CSRF 방어는 **SameSite**·CSRF 토큰과 함께 설계.
- 저장 시 **서명/무결성**(예: HMAC) 또는 **암호화**가 필요한 경우라도, 키 관리가 클라이언트에 있으면 근본적 방어가 어려움.
- **XSS 예방**: CSP(Content Security Policy), 정적 분석, 출력 인코딩, 라이브러리 업데이트.

---

## 6) 데이터 일관성/동기화 팁

- **버전 필드**를 포함하여 스키마 변경 시 마이그레이션 처리:

```ts
const APP_KEY = "app:v2";
const data = { version: 2, prefs: { theme: "dark" } };
localStorage.setItem(APP_KEY, JSON.stringify(data));
```

- 여러 탭 동기화는 **`storage` 이벤트** 활용:

```ts
window.addEventListener("storage", (e) => {
  if (e.key === "theme") applyTheme(e.newValue);
});
```

---

## 7) 무엇을 언제 쓰나

- **localStorage**: 사용성 설정(테마/언어), 비민감 **캐시성 정보**(최신 글 ID 등).
- **sessionStorage**: **탭 한정** 임시 상태(폼 복구, 검색 조건), **세션 종료와 함께 삭제되어야** 하는 데이터.

> 대용량·트랜잭션성·인덱싱이 필요한 클라이언트 저장에는 **IndexedDB**(또는 위에 얹은 Dexie/idb 라이브러리)를 사용하십시오.

---

## 8) 체크리스트

- [ ] 민감 정보 저장 금지(토큰·PII). 인증은 **HttpOnly 쿠키**로.
- [ ] 필요한 **최소 데이터**만 저장, 만료 정책(타임스탬프) 포함.
- [ ] **직렬화/버전** 전략 수립, 역직렬화 오류 대비.
- [ ] 여러 탭 동기화 필요 시 `storage` 이벤트 처리.
- [ ] 대량 데이터는 IndexedDB로 이동.
- [ ] XSS 방어(CSP 등) 및 에러 핸들링(try/catch) 구현.

---

## 결론

- `localStorage`는 **영속·출처 전역 공유**, `sessionStorage`는 **탭 한정·세션 수명**.
- 두 저장소 모두 **보안·용량·동기성** 한계를 가지므로, **민감 데이터 저장을 피하고** 사용 목적에 맞춘 경량 캐시로 활용하는 것이 바람직합니다. 장기·대용량 저장 또는 복잡한 질의가 필요하다면 **IndexedDB**를 채택하십시오.
