# React key로 컴포넌트 상태 초기화 정리

## 핵심 요약

- entity id를 key로 사용하면 대상이 바뀔 때 이전 편집 draft와 validation state가 새 인스턴스로 넘어가지 않는다.
- 페이지 전체 wrapper의 key를 바꾸면 입력 포커스, 스크롤 위치, 준비된 자식 데이터까지 불필요하게 사라진다.
- 재설정하려는 상태를 소유한 가장 작은 컴포넌트에 안정적인 userId나 documentId key를 부여한다.

## 개념 설명

React의 `key`를 바꾸면 같은 위치의 컴포넌트라도 이전 인스턴스를 버리고 새 상태로 다시 만든다.

Fiber 재조정은 type과 key가 같을 때 상태를 보존하므로, 상세 화면처럼 대상이 바뀔 때 key로 state reset 경계를 명시한다.

## 예시

```tsx
function ProfilePage({ userId }: { userId: string }) {
  return <ProfileEditor key={userId} userId={userId} />;
}
```

userId가 바뀌면 이전 사용자의 입력 draft가 새 사용자 편집 화면으로 새지 않는다.

## 면접 답변 예시

> props 변경을 감지해 여러 state를 수동 초기화하는 effect보다 컴포넌트 정체성 규칙을 한곳에 표현할 수 있다. key 변경을 서버 데이터 refetch 수단으로 쓰면 캐시 무효화 책임과 UI 인스턴스 교체 책임이 뒤섞인다. 통합 테스트에서 id 전환 후 필드가 초기화되고 이전 effect가 cleanup되며 새 input에 포커스가 유지되는지 확인한다.

## 장점

- remount 과정에서 effect cleanup이 실행되므로 이전 대상의 구독과 타이머도 state reset 경계와 함께 정리된다.

## 단점

- Math.random 같은 불안정한 key는 모든 렌더를 remount로 바꿔 로컬 상태와 DOM 재사용을 계속 파괴한다.

## 주의사항 / 실무 팁

- 대상 전환 전에 저장되지 않은 변경이 있다면 key를 바꾸기 전 확인 절차를 두어 입력 손실을 막는다.
