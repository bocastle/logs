# TypeScript Template Literal Types 정리

## 핵심 요약

- Template Literal Types로 `user:created` 같은 이벤트 이름 규칙을 표현하면 오타를 호출 지점에서 잡는다.
- 큰 union 여러 개를 교차 조합하면 생성되는 문자열 멤버 수가 폭증해 언어 서비스가 느려진다.
- 조합 축은 실제로 유한한 literal union으로 제한하고 값 종류가 크면 branded string과 parser를 사용한다.

## 개념 설명

Template Literal Types는 문자열 literal을 조합해 허용되는 문자열 패턴을 타입으로 표현한다.

union과 함께 쓰면 이벤트 이름, CSS token, route key처럼 규칙 있는 문자열을 컴파일 단계에서 제한한다.

## 예시

```ts
type Locale = "ko" | "en";
type MessageKey = `${Locale}.profile.title`;
const key: MessageKey = "ko.profile.title";
```

번역 키의 prefix와 suffix 규칙이 타입에 남아 오타를 빨리 찾는다.

## 면접 답변 예시

> MessageKey가 도메인과 동작 union에서 파생되면 새 도메인 추가 시 허용 번역 key가 함께 갱신된다. 모든 자유 문자열을 정교한 패턴으로 제한하면 확장 가능한 플러그인 key와 사용자 정의 값을 막을 수 있다. 대표 허용 문자열과 철자, 구분자, 대소문자가 틀린 실패 문자열을 타입 테스트에 함께 둔다.

## 장점

- 문자열 패턴에서 prefix를 추론해 이벤트 이름에 맞는 payload 타입을 연결할 수 있다.

## 단점

- 타입이 허용하는 문자열이어도 서버나 설정 파일에서 들어온 런타임 값의 형식까지 검증되지는 않는다.

## 주의사항 / 실무 팁

- 상수 배열에서 prefix와 suffix union을 파생해 런타임 목록과 타입 목록이 따로 변하지 않게 한다.
