# inert 속성으로 모달 배경 상호작용 막는 법 정리

## 핵심 요약

- 모달 뒤로 포커스가 새는 문제를 줄인다.
- 해제를 놓치면 페이지가 계속 비활성 상태로 남는다.
- 가능하면 native `dialog.showModal()`을 우선한다.

## 개념 설명

모달 배경의 `inert`는 대화상자 밖 DOM 하위 트리를 포커스, click, 텍스트 선택, 접근성 탐색 대상에서 제외하는 방법이다.

custom modal을 열 때 main 콘텐츠에 `inert = true`를 설정하고 닫기·예외·라우팅 경로 모두에서 제거한다. native `showModal()`은 바깥을 자동으로 inert 처리한다.

## 예시

```js
main.inert = true;
modal.hidden = false;
closeButton.addEventListener("click", () => {
  main.inert = false;
  opener.focus();
});
```

custom modal의 배경을 막고 닫을 때 `inert`를 풀며 opener로 포커스를 복귀한다.

## 면접 답변 예시

> 복잡한 focus trap 코드의 일부를 줄인다. modal 자체를 inert 조상 안에 두면 상호작용할 수 없다. Esc, 버튼, 라우팅, 예외 경로의 cleanup을 테스트한다.

## 장점

- pointer와 보조 기술 상호작용을 같이 막는다.

## 단점

- 중첩 모달에서 boolean만 바꾸면 먼저 연 배경을 너무 일찍 풀 수 있다.

## 주의사항 / 실무 팁

- custom modal은 open count로 inert 해제 시점을 관리한다.
