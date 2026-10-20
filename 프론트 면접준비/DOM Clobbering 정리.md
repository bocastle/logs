# DOM Clobbering 정리

## 핵심 요약

- DOM 기반 XSS와 설정 오염 위험을 설명할 수 있다.
- 오래된 코드의 암묵적 전역 의존을 찾기 어렵다.
- 설정 객체는 모듈 스코프 변수로 import하고 window 전역을 피한다.

## 개념 설명

DOM Clobbering은 공격자가 `id`나 `name` 속성으로 전역 객체나 속성 접근을 오염시켜 스크립트가 의도하지 않은 DOM 요소를 참조하게 만드는 취약점이다.

브라우저는 일부 요소 이름을 `window.foo`나 form property처럼 노출할 수 있으므로, 신뢰할 수 없는 HTML과 전역 변수 접근이 만나면 보안 문제가 된다.

## 예시

```html
<form id="config"></form>
<script>
  // 위험: window.config가 안전한 설정 객체라고 가정
  load(window.config.apiBase);
</script>
```

`window.config` 같은 전역 이름을 신뢰하면 삽입된 DOM 요소가 설정 객체를 가로챌 수 있다.

## 면접 답변 예시

> DOM Clobbering은 공격자가 `id`나 `name`으로 `window.config`나 form property 같은 암묵적 접근을 DOM element로 바꾸는 취약점입니다. 설정은 global named property에서 읽지 않고 module-scope import와 명시적 object를 사용하겠습니다. DOM을 직접 찾아야 한다면 selector 결과의 element type과 예상 container를 검증해야 하며 `querySelector` 자체가 신뢰를 만들어 주는 것은 아닙니다. Untrusted HTML sanitizer에는 element·attribute뿐 아니라 위험한 `id`와 `name` 정책을 포함하고 관련 regression payload를 테스트합니다.

## 장점

- 전역 접근을 줄이는 코드 리뷰 기준을 만들 수 있다.

## 단점

- 폼 속성과 전역 이름 충돌이 브라우저별로 다를 수 있다.

## 주의사항 / 실무 팁

- 신뢰할 수 없는 HTML의 id와 name을 제한한다.
