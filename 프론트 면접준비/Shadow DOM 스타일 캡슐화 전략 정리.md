# Shadow DOM 스타일 캡슐화 전략 정리

## 핵심 요약

- 웹 컴포넌트 스타일 충돌을 줄인다.
- 전역 reset과 폰트 규칙이 내부에 자동 적용되지 않는다.
- 외부에서 바꿀 수 있는 part 이름을 문서화한다.

## 개념 설명

Shadow DOM 스타일 캡슐화는 컴포넌트 내부 스타일이 바깥 CSS와 충돌하지 않게 boundary를 만드는 방식이다.

Shadow root 내부 스타일은 바깥 선택자가 직접 침투하지 못하지만, CSS custom property, `::part`, `::slotted`로 제한된 커스터마이징 지점을 열 수 있다.

## 예시

```css
/* shadow root 내부 */
:host { display: block; color: var(--panel-fg); }

/* 컴포넌트 consumer의 외부 stylesheet */
my-panel { --panel-fg: #202124; }
my-panel::part(close-button) { border-radius: 999px; }
```

내부 기본 스타일은 보호하고 host가 상속받는 token과 공개한 close-button part만 외부에서 조정하게 한다.

## 면접 답변 예시

> Shadow DOM은 내부 selector와 외부 CSS의 충돌을 줄이지만 global reset과 font rule이 모두 자동으로 적용되는 것은 아닙니다. Theme token은 custom property로 주입하고 외부에서 바꿔도 되는 내부 element만 `part`로 공개하겠습니다. `::part()`는 consumer가 `host::part(name)`으로 선택하는 API이고 `::slotted()`도 배정된 element 자체만 제한적으로 다룹니다. Style 캡슐화가 semantic과 접근성을 보장하지는 않으므로 slot content와 keyboard 동작은 별도로 검증합니다.

## 장점

- 공개 커스터마이징 지점을 API처럼 관리할 수 있다.

## 단점

- 너무 많은 part를 열면 캡슐화 이점이 줄어든다.

## 주의사항 / 실무 팁

- 토큰은 custom property로 주입한다.
