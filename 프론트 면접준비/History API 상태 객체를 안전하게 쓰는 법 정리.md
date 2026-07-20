# History API 상태 객체를 안전하게 쓰는 법 정리

## 핵심 요약

- History state는 각 방문 entry에 뒤로가기 복원용 작은 데이터를 연결하는 기능이지 영구 저장소가 아니다.
- Structured serializable 값만 넣고 대형 응답, 함수와 DOM node는 저장하지 않는다.
- 브라우저가 state를 디스크에 저장할 수 있으므로 비밀번호와 token 같은 민감정보를 넣지 않는다.

## 개념 설명

`pushState()`와 `replaceState()`의 state는 현재 history entry에 연결되고 `popstate`에서 복원할 수 있는 작은 데이터다. 목록 스크롤 위치, 선택한 탭과 resource ID처럼 다시 계산할 수 있는 UI metadata에 적합하다.

State는 structured serialization이 가능한 값이어야 하며 함수와 DOM node가 섞이면 `DataCloneError`가 발생한다. 브라우저마다 저장 크기 제한이 다르고 session 복원을 위해 디스크에 기록할 수도 있다. 따라서 서버 응답 전체나 인증정보를 담지 않고 직접 URL 방문 때도 화면을 만들 수 있어야 한다.

## 예시

```ts
try {
  history.pushState({ productId, scrollY: window.scrollY }, "", `/products/${productId}`);
} catch (error) {
  if (error instanceof DOMException && error.name === "DataCloneError") {
    history.pushState({ productId }, "", `/products/${productId}`);
  }
}
```

예외로 fallback하는 것보다 저장 전에 state schema를 작게 정의하는 편이 좋다. URL에는 resource를 다시 조회할 식별자를 남기고 state는 스크롤 위치 같은 선택적 복원 정보만 보완한다.

## 면접 답변 예시

> History state는 각 방문 entry에 뒤로가기 복원용 작은 데이터를 붙이는 기능입니다. Resource ID와 스크롤 위치처럼 structured serialization이 가능한 최소 정보만 넣고 함수, DOM node와 서버 응답 전체는 저장하지 않겠습니다. 브라우저가 state를 디스크에 보존할 수 있어 민감정보도 넣으면 안 됩니다. 직접 URL로 진입하거나 `history.state`가 null인 경우에도 화면을 만들고, `popstate`에서는 선택적인 UI 상태만 복원합니다.

## 장점

- 뒤로가기 시 목록 위치와 열린 탭 같은 UI 맥락을 빠르게 되살릴 수 있다.
- URL에 모두 표현하기 어려운 entry별 보조 상태를 연결할 수 있다.

## 단점

- State가 크면 entry마다 직렬화와 저장 비용이 누적되고 브라우저 제한을 넘을 수 있다.
- State만 믿으면 직접 URL 방문, 새 탭과 null state에서 필수 데이터가 없다.

## 주의사항 / 실무 팁

- 초기 진입과 새 탭에서는 `history.state`가 null인 경우를 먼저 처리한다.
- State schema와 크기를 제한하고 민감정보가 들어가지 않는 테스트를 둔다.
