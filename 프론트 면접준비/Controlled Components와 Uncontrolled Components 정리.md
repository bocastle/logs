# Controlled Components와 Uncontrolled Components 정리

## 핵심 요약

- `Controlled Component`는 입력값을 React state가 관리하는 방식이고, `Uncontrolled Component`는 DOM이 값을 직접 들고 있는 방식이다.
- 면접에서는 두 방식의 차이, 언제 어떤 방식을 고르는지, 폼 라이브러리가 왜 둘을 섞어 쓰는지까지 말하면 좋다.
- 일반적인 입력 검증과 UI 동기화는 controlled 쪽이 설명하기 쉽고, 성능이나 간단한 폼은 uncontrolled이 더 편할 때가 있다.
- 중요한 건 어느 쪽이 무조건 더 낫다가 아니라, 상태를 어디에 두는지가 다르다는 점이다.

## Controlled Component란

입력값을 React state로 관리하는 방식이다.  
즉 `input`의 값이 화면에 보이는 기준도 state이고, 변경도 `onChange`를 통해 state를 갱신하면서 일어난다.

이 방식의 장점은 값의 흐름이 명확하다는 점이다.  
현재 입력값이 항상 state에 있으니 검증, 조건부 렌더링, 버튼 활성화, 다른 입력과의 연동이 쉽다.

## Uncontrolled Component란

입력값을 DOM이 직접 관리하는 방식이다.  
React는 초기값 정도만 넣고, 실제 현재 값은 필요할 때 `ref`로 읽어 온다.

전통적인 HTML 폼에 더 가까운 방식이라고 보면 이해가 쉽다.  
모든 입력을 매 타이핑마다 state에 연결하지 않아도 돼서 간단한 폼에서는 오히려 부담이 적다.

## 둘의 차이를 어떻게 말하면 좋나

### 상태의 진짜 소유자

- Controlled: React state
- Uncontrolled: DOM element

이 한 문장으로 차이를 정리할 수 있다.

### 값 변경 흐름

Controlled는 사용자가 입력하면 이벤트가 발생하고, React state가 바뀐 뒤 다시 렌더링되면서 값이 반영된다.  
Uncontrolled는 사용자가 입력한 값이 DOM에 바로 들어가고, React는 필요할 때만 읽는다.

### 검증과 동기화

실시간 검증, 입력 마스킹, 다른 필드와의 연동은 controlled가 더 자연스럽다.  
반대로 제출 시점에만 값이 필요하다면 uncontrolled도 충분하다.

## 언제 Controlled를 쓰면 좋나

### 실시간 검증이 필요한 경우

비밀번호 규칙 안내, 에러 메시지 표시, 버튼 활성화 조건처럼 입력값에 따라 UI가 바로 바뀌면 controlled가 편하다.

### 여러 입력값이 서로 영향을 주는 경우

예를 들어 국가를 바꾸면 전화번호 형식이 달라지거나, 체크박스 선택에 따라 추가 필드가 열리면 state 기반 관리가 더 명확하다.

## 언제 Uncontrolled가 더 낫나

### 단순한 폼을 빠르게 처리할 때

검색창 하나, 업로드 input, 제출 시점에만 값이 필요한 폼은 uncontrolled로도 충분하다.

### 파일 입력처럼 원래 DOM 중심인 요소

`file input`은 대표적으로 uncontrolled에 가깝게 다루는 경우가 많다.  
브라우저 제약 때문에 값 제어가 제한적이라, ref로 접근하는 패턴을 함께 알고 있으면 좋다.

## 폼 라이브러리는 왜 이걸 섞어 쓰나

실무에서는 React Hook Form 같은 라이브러리가 자주 언급된다.  
이런 라이브러리는 불필요한 렌더를 줄이기 위해 uncontrolled 기반으로 동작하는 경우가 많고, 복잡한 커스텀 입력은 controlled 방식으로 연결하기도 한다.

즉 실무 답변에서는 "둘 중 하나만 옳다"보다, 요구사항에 따라 혼합할 수 있다고 말하는 편이 더 현실적이다.

## 면접 답변 예시

> Controlled Component는 입력값을 React state가 관리하는 방식이고, Uncontrolled Component는 DOM이 입력값을 직접 들고 있는 방식입니다. Controlled는 값의 흐름이 명확해서 실시간 검증이나 다른 UI와의 동기화에 유리하고, Uncontrolled는 간단한 폼이나 성능 부담을 줄이고 싶을 때 더 가볍게 쓸 수 있습니다. 실무에서는 둘 중 하나만 고집하기보다, 폼 성격에 따라 섞어서 사용하는 경우도 많습니다.

## 장점

- controlled는 디버깅과 상태 추적이 쉽다.
- uncontrolled는 단순한 입력 처리에서 코드가 가벼울 수 있다.
- 두 방식을 구분해서 설명하면 React 폼 설계를 깊이 있게 이해한 인상을 줄 수 있다.

## 단점

- controlled는 입력마다 렌더링이 연결돼 부담이 될 수 있다.
- uncontrolled는 현재 값을 추적하거나 실시간 검증하기가 상대적으로 불편하다.
- 팀 내 기준 없이 섞으면 오히려 폼 로직이 복잡해질 수 있다.

## 주의사항 / 실무 팁

- 면접에서는 "상태를 누가 소유하느냐"를 먼저 말하면 설명이 정리된다.
- 파일 업로드 input은 controlled처럼 완전히 다루기 어렵다는 점을 같이 기억해 두면 좋다.
- 성능 때문에 무조건 uncontrolled을 고르기보다, 실제 병목인지 먼저 확인하는 게 맞다.
- 폼 라이브러리 경험이 있다면 controlled와 uncontrolled을 어떻게 조합했는지 사례를 붙이면 답변이 더 설득력 있다.
