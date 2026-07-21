# URL 상태와 React 상태를 나누는 기준 정리

## 핵심 요약

- 검색 조건을 URL에 두면 새로고침, 공유 링크, 뒤로 가기에서도 같은 결과 화면을 복원할 수 있다.
- 토큰이나 개인 메모를 query string에 넣으면 브라우저 기록, 로그, referrer를 통해 민감한 값이 노출될 수 있다.
- query를 읽을 때 허용 값, 기본값, 숫자 범위를 한 파서에서 검증하고 잘못된 주소는 canonical URL로 교정한다.

## 개념 설명

URL 상태는 검색 조건, 페이지 번호처럼 공유·복원되어야 하는 값을 주소에 두고, React 상태는 입력 중인 draft나 열린 패널처럼 화면 내부 생명주기에 묶인 값을 둔다.

라우터가 query string을 읽어 서버 요청과 history를 만들고, 컴포넌트는 `useState`로 즉시 반응해야 하는 임시 상태를 관리한다.

## 예시

```tsx
const params = new URLSearchParams(location.search);
const [draft, setDraft] = useState(params.get("q") ?? "");
function submit() {
  router.replace(`/search?q=${encodeURIComponent(draft)}`);
}
```

검색어 draft는 빠르게 바뀌지만 제출된 q는 URL에 남아 새로고침과 공유 링크에서 같은 결과를 만든다.

## 면접 답변 예시

> 입력 중인 draft를 useState에 남기면 타이핑마다 history 항목을 만들지 않고 즉시 반응하는 편집 경험을 유지한다. 모든 키 입력에 history.push를 호출하면 사용자가 한 번의 검색을 되돌리려고 뒤로 가기를 여러 번 눌러야 한다. 빈 URL에서의 초기값, 직접 연 deep link, 연속 제출 뒤 브라우저 앞뒤 이동을 라우터 통합 테스트로 확인한다.

## 장점

- 제출된 query가 주소에 남으므로 서버 loader와 클라이언트 캐시가 동일한 검색 key를 사용할 수 있다.

## 단점

- 같은 검색어를 URLSearchParams와 컴포넌트 state에서 독립적으로 갱신하면 두 원본이 엇갈리는 동기화 버그가 생긴다.

## 주의사항 / 실무 팁

- 필터 조정처럼 현재 기록을 다듬는 변경은 replace를, 사용자가 확정한 탐색 단계는 push를 사용하도록 history 정책을 나눈다.
