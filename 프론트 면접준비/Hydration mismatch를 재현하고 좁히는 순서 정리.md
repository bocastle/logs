# Hydration mismatch를 재현하고 좁히는 순서 정리

## 핵심 요약

- Hydration mismatch를 같은 production 조건에서 재현하면 개발 모드의 추가 렌더링이나 확장 프로그램 영향과 실제 SSR 차이를 분리할 수 있다.
- 개발 서버에서만 확인하면 Strict Mode 재렌더나 hot reload 흔적을 hydration mismatch 원인으로 착각할 수 있다.
- 먼저 production SSR HTML을 저장하고 같은 URL, 쿠키, locale, timezone으로 클라이언트 hydration을 반복해 재현성을 확인한다.

## 개념 설명

Hydration mismatch 재현과 축소는 서버 HTML과 클라이언트 첫 렌더 결과가 달라지는 지점을 안정적으로 다시 만들고 차이를 만든 입력을 하나씩 제거하는 디버깅 절차다.

production build SSR 결과, 브라우저 hydration 경고, locale·time·random·user agent·storage 같은 비결정 입력을 같은 조건으로 고정한 뒤 의심 컴포넌트를 작은 boundary로 격리한다.

## 예시

```tsx
// 서버와 클라이언트 첫 렌더에서 값이 달라질 수 있는 코드
const label = new Date().toLocaleString();
return <time>{label}</time>;
```

서버 HTML의 time 문자열과 클라이언트 hydration 시점의 문자열이 달라지면 mismatch가 재현되므로 날짜 값을 props로 고정하거나 client-only 렌더로 분리한다.

## 면접 답변 예시

> 의심 영역을 작은 Suspense 또는 client boundary로 나누면 전체 페이지가 아니라 문제가 생긴 컴포넌트 계약만 검증하게 된다. 브라우저 전용 API를 렌더 중 바로 읽으면 서버에서는 없는 값이 클라이언트 첫 렌더에 섞여 mismatch 범위가 넓어진다. 원인을 찾은 뒤에는 서버에서 동일 값을 내려주거나 useEffect 이후 client-only로 바꾸고, hydration 경고가 사라지는지 회귀 테스트를 남긴다.

## 장점

- 서버 HTML snapshot과 클라이언트 첫 렌더 값을 나란히 보면 locale, 시간, 랜덤값처럼 결정적이지 않은 입력을 빠르게 좁힐 수 있다.

## 단점

- 경고를 숨기려고 suppressHydrationWarning을 넓게 쓰면 서버 HTML과 클라이언트 상태가 실제로 다른 버그를 놓친다.

## 주의사항 / 실무 팁

- Date, Math.random, window 크기, localStorage, 권한 상태처럼 서버와 클라이언트가 다르게 볼 입력을 목록화해 하나씩 고정한다.
