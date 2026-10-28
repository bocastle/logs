# Selection API와 에디터 구현 정리

## 핵심 요약

- 브라우저가 전달하는 편집 의도를 undo 모델과 연결할 수 있다.
- composition 중 재렌더링하면 한글 조합 문자열이 중복되거나 사라질 수 있다.
- bookmark는 블록 id와 text offset 같은 모델 좌표로 저장한다.

## 개념 설명

Selection API 기반 에디터 구현은 caret과 선택 범위를 모델 상태, DOM 재렌더링, IME composition 사이에서 잃지 않도록 좌표를 저장하고 복원하는 작업이다.

`beforeinput`의 inputType으로 편집 의도를 받고 composition 중에는 DOM을 강제 교체하지 않으며, 렌더 뒤 Selection에 새 Range를 넣어 caret을 복원한다.

## 예시

```ts
editor.addEventListener("beforeinput", (event) => {
  if (isComposing) return;
  const bookmark = saveSelection(editor, window.getSelection());
  applyEdit(event.inputType, event.data);
  restoreSelection(editor, bookmark);
});
editor.addEventListener("compositionstart", () => { isComposing = true; });
editor.addEventListener("compositionend", () => { isComposing = false; });
```

beforeinput에서 편집 의도와 Selection bookmark를 연결하고 composition 구간의 중간 DOM을 덮어쓰지 않는다. bookmark는 node 참조보다 문서 모델 위치가 안정적이다.

## 면접 답변 예시

> IME와 일반 키 입력을 다른 갱신 시점으로 다룰 수 있다. selectionchange를 모든 상태 변경으로 처리하면 이벤트 루프가 생길 수 있다. beforeinput inputType별 undo, paste, delete 시나리오를 실제 브라우저에서 테스트한다.

## 장점

- caret 복원으로 포맷 적용 뒤 입력 위치가 튀는 일을 줄인다.

## 단점

- DOM node 참조만 저장하면 normalization 뒤 복원할 수 없다.

## 주의사항 / 실무 팁

- compositionstart부터 compositionend까지 외부 렌더를 지연한다.
