# React Portal 접근성 정리

## 핵심 요약

- createPortal은 모달을 overflow와 stacking context 밖에 렌더링하면서 React Context와 event 연결은 유지한다.
- 모달이 열린 뒤 포커스를 가두지 않으면 키보드 사용자가 가려진 배경 컨트롤로 이동할 수 있다.
- 열릴 때 첫 의미 있는 컨트롤로 포커스를 옮기고 닫힐 때 trigger로 복원하며 배경에는 inert를 적용한다.

## 개념 설명

React Portal은 DOM 위치를 다른 컨테이너로 옮겨 렌더링하지만 React 이벤트와 상태 흐름은 기존 트리에 남긴다.

모달이나 팝오버를 portal로 띄울 때 focus trap, aria-modal, 배경 inert 처리를 함께 해야 접근성 흐름이 맞는다.

## 예시

```tsx
return createPortal(
  <div role="dialog" aria-modal="true"><ModalBody /></div>,
  document.body,
);
```

시각적으로는 body 아래에 있지만 모달 의미와 포커스 제한을 명시해야 탐색이 새지 않는다.

## 면접 답변 예시

> aria-modal과 이름을 갖춘 dialog를 portal에 두면 시각적 레이어와 보조기술의 현재 작업 범위를 맞출 수 있다. SSR에 portal container가 없거나 hydration 전에 다른 위치를 선택하면 서버 markup과 클라이언트 구조가 달라진다. outside interaction은 event target의 composedPath와 overlay 경계를 확인해 portal 내부 클릭을 구분한다.

## 장점

- overlay를 공통 root에 모으면 z-index 순서와 scroll lock 정책을 애플리케이션 전체에서 일관되게 관리한다.

## 단점

- DOM 부모만 기준으로 outside click을 판정하면 React 트리에서 전파된 portal 이벤트를 잘못 닫기 동작으로 처리한다.

## 주의사항 / 실무 팁

- dialog에 role, aria-labelledby, 필요 시 aria-describedby를 부여하고 Escape 및 Tab 순환을 키보드로 테스트한다.
