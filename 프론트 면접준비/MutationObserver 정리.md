# MutationObserver 정리

## 핵심 요약

- `MutationObserver`는 DOM 변경을 mutation record 묶음으로 비동기 전달하는 API다.
- React state로 이미 아는 변경보다 외부 script·Web Component처럼 직접 소유하지 않은 DOM 경계를 관찰할 때 적합하다.
- 관찰 root와 `childList`·`attributes`·`subtree` 옵션을 최소화하고 cleanup에서 `disconnect()`를 호출한다.

## 개념 설명

`MutationObserver`는 DOM node 추가·삭제와 attribute·text 변경을 `MutationRecord` 목록으로 전달한다. 변경 직후 동기 event를 발생시키는 옛 mutation event와 달리 여러 변경을 묶어 callback에서 처리한다.

Observer는 `observe()`의 대상과 option 범위에서만 기록한다. Document 전체에 `subtree`와 모든 attribute를 켜면 관련 없는 변경까지 쌓인다. Callback 안에서 다시 같은 DOM을 수정하면 다음 record를 계속 만들 수 있으므로 원인 mutation을 필터링하고 rendering 작업은 필요하면 다음 frame으로 넘긴다.

## 예시

```ts
const observer = new MutationObserver((records) => {
  for (const record of records) {
    if (record.type === "childList") refreshEmptyState();
  }
});
observer.observe(list, { childList: true, subtree: false });
```

목록 바로 아래 자식 추가·삭제만 관찰한다. Component가 사라질 때 `disconnect()`하고 아직 처리하지 않은 record를 버릴지 `takeRecords()`로 처리할지 수명 정책을 정한다.

## 면접 답변 예시

> `MutationObserver`는 DOM 변경을 record 묶음으로 비동기 전달하는 API입니다. React state로 이미 아는 변경보다 외부 script나 Web Component가 바꾸는 DOM 경계에 사용하겠습니다. 관찰 root와 mutation type을 필요한 범위로 제한하고 callback이 같은 DOM을 다시 바꿔 loop를 만들지 않게 합니다. Component cleanup에서는 `disconnect()`를 호출하고 남은 record 처리 정책도 정합니다.

## 장점

- Deprecated mutation event보다 여러 변경을 묶는 안전한 비동기 관찰 모델을 사용한다.
- 직접 제어하지 않는 DOM 통합 지점의 변화를 감지할 수 있다.

## 단점

- 관찰 범위가 넓으면 관련 없는 record 처리와 memory 비용이 커진다.
- Callback이 같은 조건의 DOM을 다시 바꾸면 관찰 loop가 생길 수 있다.

## 주의사항 / 실무 팁

- Callback에서는 record type과 target을 먼저 거르고 rendering 계산은 필요하면 다음 frame으로 넘긴다.
- Unmount·route 전환에서 `disconnect()`와 pending record 처리를 테스트한다.
