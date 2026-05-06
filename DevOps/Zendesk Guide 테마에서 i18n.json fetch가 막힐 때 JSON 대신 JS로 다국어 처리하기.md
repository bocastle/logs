# Zendesk Guide 테마에서 i18n.json fetch가 막힐 때: JSON 대신 JS로 다국어 처리하기

Zendesk Guide(Help Center) 테마 작업을 하다 보면 번역 리소스를 `en.json`, `ja.json` 같은 파일로 두고 `fetch()`로 읽어오려는 시도를 하게 됩니다. 그런데 실제 운영 환경에서는 다음과 같은 이유로 `fetch`가 실패하는 경우가 있습니다.

- CSP(콘텐츠 보안 정책)로 요청이 차단됨
- 에셋 URL은 나오지만 실제 응답이 403/404로 떨어짐
- 테마 편집기에서 폴더 구조(`assets/i18n/...`)를 기대했는데 파일 단위만 관리되어 경로 설계가 꼬임

이 글에서는 “통신이 막혀서 JSON을 못 가져오는 상황”에서 가장 안정적으로 다국어를 적용하는 방법인 **JSON 대신 JS 파일로 번역 테이블을 포함**하는 방식을 정리합니다.

## 문제: `{{asset 'en.json'}}`는 JSON이 아니라 URL이다

테마에서 아래 코드를 작성하면, 많은 분들이 `assetsEn`에 JSON 객체가 들어올 것으로 기대합니다.

```js
var assetsEn = {{asset 'en.json'}};
console.log("enJson", assetsEn);
```

하지만 `{{asset 'en.json'}}`는 **파일 내용이 아니라 에셋의 URL 문자열을 출력**합니다. 즉 `assetsEn`은 JSON이 아니라 `"https://.../en.json"` 같은 문자열이 됩니다. 보통은 그 URL을 다시 `fetch()`로 읽어와야 합니다.

```js
const enUrl = "{{asset 'en.json'}}";
const res = await fetch(enUrl);
const enJson = await res.json();
```

그런데 여기서 CSP/CORS/권한/리다이렉트 등으로 `fetch`가 막히면 더 이상 진행이 어렵습니다.

## 해결: JSON 파일 대신 JS로 번역 객체를 포함한다

가장 확실한 우회는 **번역 데이터를 JS 파일에 상수 객체로 정의**하고, 메인 스크립트에서 이를 참조하는 방식입니다. 네트워크 요청이 없어지기 때문에 `fetch`가 막히는 환경에서도 동작합니다.

핵심 아이디어는 간단합니다.

- `en.js`, `ja.js`를 에셋으로 올린다
- 각 파일에서 번역 객체를 `window`에 붙인다
- 현재 로케일을 읽어 해당 객체를 선택한다
- `t('key.path')` 형태로 문구를 꺼내서 렌더링한다

## 1) 번역 파일 만들기: en.js / ja.js

**en.js**

```js
window.I18N_EN = {
  header: {
    headerLink1: "CASTLE Homepage↗",
    headerLink2: "Partners↗",
    headerLink3: "Test",
  },
  heroTitle: "How can we help you?",
  commonSearch: "Common search",
  searchPlaceholder: "Search...",
  signinBtn: "Sign Up",
};
```

**ja.js**

```js
window.I18N_JA = {
  header: {
    headerLink1: "CASTLEホームページ↗",
    headerLink2: "パートナー↗",
    headerLink3: "テスト",
  },
  heroTitle: "お困りですか？",
  commonSearch: "よく検索されるキーワード",
  searchPlaceholder: "検索...",
  signinBtn: "会員登録",
};
```

## 2) 로케일 선택 + t() 함수 만들기

Guide 페이지의 언어는 보통 `document.documentElement.lang`로 접근할 수 있습니다. 예를 들어 `en-us`, `ja-jp` 같은 값이 들어옵니다.

```js
function getLocale() {
  const lang = (document.documentElement.lang || "en").toLowerCase();
  if (lang.startsWith("ja")) return "JA";
  return "EN";
}

function getDict() {
  const locale = getLocale();
  if (locale === "JA" && window.I18N_JA) return window.I18N_JA;
  return window.I18N_EN || {};
}

// "header.headerLink1"처럼 점 표기 경로로 접근
function t(path) {
  const dict = getDict();
  return (
    path
      .split(".")
      .reduce((acc, k) => (acc && acc[k] != null ? acc[k] : null), dict) ?? path
  );
}
```

## 3) 실제 DOM에 적용하기

```js
document.querySelector("#heroTitle").textContent = t("heroTitle");
document.querySelector("#signinBtn").textContent = t("signinBtn");
document.querySelector("#headerLink1").textContent = t("header.headerLink1");
```

## 4) 가장 중요한 체크포인트: 로딩 순서

이 방식은 `en.js`, `ja.js`가 **메인 스크립트보다 먼저 로드**되어야 합니다.

- 헤더에서 `<script src="{{asset 'en.js'}}"></script>`를 먼저 로드
- 그 다음 메인 스크립트에서 `t()` 호출

로딩 순서가 뒤집히면 `window.I18N_EN`이 아직 없어서 빈 객체가 됩니다.

## 이 방식이 좋은 경우 / 나쁜 경우

좋은 경우

- `fetch`가 CSP/CORS로 막혀 해결이 어려운 경우
- 테마에서 파일 관리가 단순하고, 빠르게 안정화가 필요한 경우
- 번역량이 크지 않고, 빌드 도구 없이 운영하고 싶은 경우

주의할 점

- 번역 데이터가 매우 크면 JS 로드 크기가 커질 수 있음
- 번역 리소스를 “동적으로 서버에서 갱신”해야 한다면 JSON fetch 방식이 더 적합

## 결론

Zendesk Guide 테마에서 `i18n.json`을 `fetch`로 가져오려다 막히는 경우, 가장 실무적으로 안전한 해결책은 **번역 데이터를 JS로 포함하는 방식**입니다. 네트워크 호출이 사라지고, 테마의 제약(폴더 없음, CSP 등)에서도 안정적으로 다국어 렌더링을 구현할 수 있습니다.
