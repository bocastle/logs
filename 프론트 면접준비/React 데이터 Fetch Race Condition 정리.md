# React 데이터 Fetch Race Condition 정리

## 핵심 요약

- AbortController로 이전 fetch를 중단하면 사용하지 않을 응답의 네트워크와 파싱 작업을 줄일 수 있다.
- AbortError를 일반 실패로 표시하면 사용자가 새 항목을 선택할 때마다 불필요한 오류 UI가 깜빡인다.
- effect 실행마다 새 controller를 만들고 cleanup에서 abort한 뒤 catch에서 signal.aborted를 먼저 확인한다.

## 개념 설명

React 데이터 Fetch Race Condition은 늦게 끝난 이전 요청이 최신 화면 상태를 덮어쓰는 문제다.

요청마다 AbortController나 sequence id를 두고 cleanup에서 이전 요청을 취소하거나 결과 적용 조건을 확인한다.

## 예시

```tsx
useEffect(() => {
  const controller = new AbortController();
  void (async () => {
    try {
      const response = await fetch(`/api/users/${id}`, { signal: controller.signal });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const user = await response.json();
      if (!controller.signal.aborted) load(user);
    } catch (error) {
      if (!controller.signal.aborted) reportError(error);
    }
  })();
  return () => controller.abort();
}, [id]);
```

id가 바뀌면 이전 fetch가 취소되어 오래된 응답이 새 상세 화면을 덮지 않는다.

## 면접 답변 예시

> 요청 생명주기가 effect dependency와 맞아 컴포넌트 unmount 뒤 setState 경로도 정리된다. 하나의 controller를 여러 effect 실행에서 재사용하면 이미 abort된 signal 때문에 새 요청도 즉시 취소된다. 첫 요청을 의도적으로 늦추고 두 번째 요청을 먼저 완료하는 테스트로 최종 화면이 최신 id인지 검증한다.

## 장점

- cleanup 이후 결과 적용을 막으면 느린 이전 id 응답이 최신 상세 화면의 state를 덮지 않는다.

## 단점

- fetch가 아닌 비동기 라이브러리가 signal을 무시하면 abort 호출만으로 오래된 callback 적용을 막을 수 없다.

## 주의사항 / 실무 팁

- 취소를 지원하지 않는 작업은 증가하는 sequence id를 결과와 비교해 최신 요청만 commit하게 한다.
