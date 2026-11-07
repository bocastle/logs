# Drag and Drop API 정리

## 핵심 요약

- 브라우저 간 파일과 텍스트 드롭 흐름을 제공한다.
- touch 환경의 지원이 pointer 환경과 다를 수 있다.
- `DataTransfer.types`와 파일 MIME type을 검증한다.

## 개념 설명

HTML Drag and Drop API는 `dragstart`, `dragover`, `drop`과 `DataTransfer`로 요소 이동이나 파일 드롭 데이터를 전달하는 브라우저 API다.

drag source는 `dragstart`에서 `dataTransfer.setData()`를 설정하고 drop target은 `dragover` 기본 동작을 취소해 drop을 허용한 뒤 `drop`에서 데이터를 읽는다.

## 예시

```js
target.addEventListener("dragover", (event) => event.preventDefault());
target.addEventListener("drop", (event) => {
  const id = event.dataTransfer.getData("text/plain");
  moveItem(id);
});
```

`dragover` default를 막아 drop target을 활성화하고 `DataTransfer`의 id로 항목을 이동한다.

## 면접 답변 예시

> HTML Drag and Drop API는 `DataTransfer`로 text, URL이나 file을 source에서 drop target으로 전달하는 event model입니다. Target은 `dragover`의 default를 막아 drop을 허용하되 `types`, file 수·크기와 실제 content를 검증해야 합니다. 외부 file의 MIME과 filename은 신뢰할 수 없어 upload server에서도 다시 검사하겠습니다. Touch 지원이 환경마다 다르고 keyboard 조작을 자동 제공하지 않으므로 reorder에는 이동 button이나 keyboard shortcut과 상태 안내를 함께 둡니다.

## 장점

- drag source와 drop target의 이벤트 경계가 명확하다.

## 단점

- 외부 drop 데이터를 신뢰하면 보안 문제가 생긴다.

## 주의사항 / 실무 팁

- drop 불가 영역과 실패 상태를 화면에 표시한다.
