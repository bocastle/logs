# Client-side Routing과 History API 정리

## 핵심 요약

- 전체 문서 재로딩 없이 화면 전환 상태를 유지할 수 있다.
- 서버가 route fallback을 제공하지 않으면 새로고침이 404가 된다.
- 같은 origin의 명시된 route 링크만 인터셉트한다.

## 개념 설명

Client-side routing은 문서를 다시 받지 않고 URL과 화면을 바꾸되 브라우저의 session history 탐색 계약을 유지하는 내비게이션 방식이다.

내부 링크 클릭은 `pushState`로 새 route entry를 만들고, 사용자의 뒤로가기와 앞으로가기는 `popstate`에서 현재 pathname을 다시 해석해 화면을 렌더링한다.

## 예시

```ts
document.addEventListener("click", (event) => {
  const link = (event.target as Element).closest<HTMLAnchorElement>("a[data-route]");
  if (
    !link || event.defaultPrevented || event.button !== 0 ||
    event.metaKey || event.ctrlKey || event.shiftKey || event.altKey ||
    link.target || link.origin !== location.origin
  ) return;
  event.preventDefault();
  history.pushState(null, "", link.href);
  renderRoute(new URL(link.href).pathname);
});
addEventListener("popstate", () => renderRoute(location.pathname));
```

앱 내부 route 링크만 가로채고 history entry를 추가한다. popstate에서는 state를 변경하지 않고 현재 URL을 단일 진실 원천으로 다시 읽는다.

## 면접 답변 예시

> Client-side routing은 문서를 다시 받지 않고 URL과 화면을 바꾸면서 browser history의 뒤로가기 계약을 유지하는 방식입니다. 내부 link는 `pushState()`로 entry를 추가하고 `popstate`에서는 현재 URL을 다시 해석하되, 수정키 클릭·새 창·다른 origin link는 기본 동작을 그대로 두겠습니다. 서버도 같은 공개 route를 직접 열거나 새로고침했을 때 올바른 문서를 반환해야 합니다. 화면 전환 뒤에는 `document.title`, heading focus와 scroll 정책까지 갱신해 URL만 바뀌고 접근성 맥락이 남는 문제를 막습니다.

## 장점

- 브라우저 뒤로가기와 앞으로가기 버튼이 앱 route와 함께 동작한다.

## 단점

- 수정키 클릭과 외부 링크까지 가로채면 기본 링크 동작을 망친다.

## 주의사항 / 실무 팁

- 서버도 모든 공개 route에 올바른 문서나 응답을 제공하게 설정한다.
