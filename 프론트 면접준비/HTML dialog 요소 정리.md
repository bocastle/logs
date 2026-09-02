# HTML dialog 요소 정리

## 핵심 요약

- 대화상자 semantics를 표준 요소로 전달한다.
- `open` 속성만 직접 바꾸면 정상 열기 절차를 건너뛸 수 있다.
- `show()`과 `showModal()`을 상호작용 요구에 맞게 고른다.

## 개념 설명

HTML `dialog` 요소는 문서 안의 모달 또는 비모달 대화상자를 표현하는 시맨틱 요소다.

`show()`는 비모달, `showModal()`은 top layer 모달로 열고 `close(value)`는 `returnValue`를 남기며 닫는다. `cancel`과 `close` 이벤트의 용도가 다르다.

## 예시

```js
const dialog = document.querySelector("dialog");
dialog.showModal();
dialog.addEventListener("close", () => save(dialog.returnValue));
```

모달을 열고 닫힌 뒤 `returnValue`로 사용자의 선택을 처리한다.

## 면접 답변 예시

> HTML `dialog`는 modal과 non-modal 대화상자를 browser 표준 동작으로 표현하는 element입니다. `showModal()`은 top layer에 modal로 열고 바깥 문서를 inert하게 만들며, `show()`는 non-modal이므로 상호작용 요구에 맞게 선택해야 합니다. 중복 호출을 피하려고 열기 전에 `dialog.open`을 확인하고 Esc의 `cancel`과 실제 닫힌 뒤의 `close` event를 구분하겠습니다. Element가 제목 연결과 초기 focus 위치까지 자동으로 결정해 주는 것은 아니어서 accessible name, focus 진입과 복귀도 함께 설계합니다.

## 장점

- top layer와 Esc cancel 동작을 활용한다.

## 단점

- 중복 `showModal()` 호출은 예외를 낼 수 있다.

## 주의사항 / 실무 팁

- 닫기 상태는 `close` 이벤트에서 처리한다.
