# HTML inert 속성 정리

## 핵심 요약

- 포커스와 pointer 비활성화를 한 속성으로 표현한다.
- 상위 요소에 잘못 설정하면 전체 앱을 사용할 수 없다.
- inert 설정 주체와 해제 주체를 같은 경계에 둔다.

## 개념 설명

HTML `inert` 속성은 해당 요소와 flat tree 하위 콘텐츠를 사용자 입력과 접근성 탐색 대상에서 제외한다.

inert subtree의 요소는 focus되거나 click될 수 없고 일반적으로 accessibility tree에서도 제외된다. `aria-hidden`과 달리 실제 상호작용도 막는다.

## 예시

```html
<main inert>…</main>
<aside>지금 작업해야 하는 패널</aside>
```

임시로 비활성화할 포함 영역에 `inert`를 주고 활성 패널은 subtree 밖에 둔다.

## 면접 답변 예시

> HTML `inert`는 지정한 영역과 flat tree 하위 요소를 focus, click과 접근성 탐색 대상에서 함께 제외하는 속성입니다. Modal이나 단계형 작업에서 비활성 배경에 적용하면 `aria-hidden`만 쓸 때처럼 focus가 숨어 있는 영역으로 들어가는 문제를 줄일 수 있습니다. 다만 현재 focus가 inert 영역 안에 남지 않도록 활성 panel로 먼저 옮기고 닫을 때 원래 trigger로 복원하겠습니다. `inert`는 흐림이나 비활성 색을 자동으로 주지 않으므로 시각 상태를 별도로 설계하고 keyboard, pointer와 accessibility tree를 모두 테스트합니다.

## 장점

- 접근성 트리와 화면 상태를 맞추기 쉬워진다.

## 단점

- 활성 UI를 inert subtree 안에 두면 일반 자식의 `inert`만 해제해도 상위 비활성 상태를 벗어나지 못한다.

## 주의사항 / 실무 팁

- 상태 변경 후 키보드와 접근성 트리를 함께 확인한다.
