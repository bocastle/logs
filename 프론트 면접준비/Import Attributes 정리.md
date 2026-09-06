# Import Attributes 정리

## 핵심 요약

- 모듈 로더가 예상하지 않은 콘텐츠 타입을 실행하는 위험을 줄인다.
- 구형 assert 문법과 with 문법을 섞으면 파서 오류가 생긴다.
- target 브라우저와 번들러가 with 문법을 그대로 지원하는지 확인한다.

## 개념 설명

Import Attributes는 JavaScript 모듈을 가져올 때 리소스의 예상 타입 같은 로더 조건을 `with` 절에 선언하는 문법이다.

JSON module은 `with { type: 'json' }`을 붙여 요청 의도와 응답 MIME 검사를 연결하며 정적 import와 동적 import options 양쪽에서 타입을 명시할 수 있다.

## 예시

```js
import config from "./config.json" with { type: "json" };

const labels = await import("./labels.ko.json", {
  with: { type: "json" },
});
console.log(config.apiBase, labels.default.save);
```

정적 JSON import와 동적 JSON import에 같은 type attribute를 사용한다. 서버는 JSON에 맞는 MIME type으로 응답해야 한다.

## 면접 답변 예시

> 정적·동적 import에서 같은 리소스 타입 계약을 사용할 수 있다. 도구가 attribute를 제거하거나 변환하면 런타임 동작과 달라질 수 있다. 동적 import에서도 두 번째 options 객체의 with 구조를 일관되게 사용한다.

## 장점

- JSON 의존성이 module graph에 명시적으로 나타난다.

## 단점

- 서버 MIME 설정이 틀리면 파일 내용이 정상이어도 import가 거부된다.

## 주의사항 / 실무 팁

- JSON 경로에는 application/json MIME 응답 테스트를 둔다.
