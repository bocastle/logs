# React Custom Hook API 설계 정리

## 핵심 요약

- Custom Hook이 구독 setup과 cleanup을 감추면 여러 컴포넌트가 같은 생명주기 규칙을 재사용한다.
- 매 render 새 action 함수와 객체를 반환하면 소비자의 dependency와 memoized child가 계속 변경된다.
- 입력, 반환 상태, 오류, cleanup 책임을 작은 타입으로 정의하고 이름에 구현 기술보다 도메인 목적을 담는다.

## 개념 설명

React Custom Hook API 설계는 상태 로직을 재사용 가능한 함수로 빼되 반환값과 입력 계약을 안정적으로 정하는 일이다.

hook은 다른 hook을 조합하고, 반환 객체나 tuple의 안정성, 오류 상태, cleanup 책임을 호출자가 이해할 수 있어야 한다.

## 예시

```tsx
function useAsyncSave() {
  const [state, setState] = useState<SaveState>({ status: "idle" });
  return { state, save: useCallback(runSave, []) };
}
```

상태와 명령을 명시적으로 반환하면 사용하는 컴포넌트가 pending과 error UI를 직접 연결할 수 있다.

## 면접 답변 예시

> 도메인 동작을 useSearch나 useConnection 같은 이름으로 묶어 컴포넌트가 저수준 effect보다 사용자 의도를 말한다. 사용자 정의 Hook을 조건부로 호출할 수 있게 API를 설계하면 소비자가 Hook 순서 규칙을 위반하게 된다. StrictMode의 setup-cleanup-setup 순서와 unmount 중 요청 취소를 테스트해 resource 누수를 확인한다.

## 장점

- pending, data, error와 명령을 명시적 반환값으로 제공해 소비자가 화면 상태를 빠짐없이 렌더링한다.

## 단점

- hook 안에서 navigation이나 toast를 암묵적으로 실행하면 호출부만 읽고 부작용 시점을 예측하기 어렵다.

## 주의사항 / 실무 팁

- 함수 참조 안정성이 실제 소비 계약일 때만 useCallback을 적용하고 그 보장을 테스트나 문서에 명시한다.
