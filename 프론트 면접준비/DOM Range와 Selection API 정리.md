# DOM Range와 Selection API 정리

## 핵심 요약

- 선택한 문서 조각의 텍스트와 DOM 구조를 함께 읽을 수 있다.
- rangeCount 확인 없이 getRangeAt을 호출하면 예외가 발생한다.
- Selection의 collapsed와 rangeCount를 먼저 검사한다.

## 개념 설명

DOM Range는 문서 안의 시작·끝 boundary point를 나타내고 Selection은 사용자가 현재 선택한 하나 이상의 Range와 방향 정보를 보관하는 API다.

`window.getSelection`으로 Selection을 얻고 rangeCount를 확인한 뒤 `getRangeAt`으로 Range를 읽어 텍스트, 복제한 fragment, 화면 좌표를 구할 수 있다.

## 예시

```ts
const selection = window.getSelection();
if (selection && selection.rangeCount > 0 && !selection.isCollapsed) {
  const range = selection.getRangeAt(0);
  const excerpt = range.cloneContents().textContent ?? "";
  showQuoteToolbar(range.getBoundingClientRect(), excerpt);
}
```

Selection이 비어 있지 않을 때 첫 Range의 내용과 위치를 읽는다. 읽기만 하는 동작에서도 selection이 에디터 영역 안에 있는지 확인해야 한다.

## 면접 답변 예시

> DOM Range는 start·end node와 offset으로 문서 안의 정확한 범위를 나타내고 Selection은 사용자가 선택한 range를 보관합니다. `rangeCount`와 collapsed 상태를 먼저 확인하고 편집기 밖 selection은 처리하지 않겠습니다. Range는 live DOM 경계에 의존하므로 React rerender나 node 교체 뒤에는 원래 문장을 가리키지 않을 수 있습니다. 저장이 필요하면 DOM 변경 전에 안정적인 block ID와 text offset으로 변환하고 복원 시 content version도 확인합니다.

## 장점

- 선택 영역의 client rect를 기준으로 주석 UI를 배치할 수 있다.

## 단점

- 서로 다른 요소를 가로지르는 Range는 예상보다 복잡한 fragment를 만든다.

## 주의사항 / 실무 팁

- 기능 영역이 selection의 commonAncestorContainer를 포함하는지 확인한다.
