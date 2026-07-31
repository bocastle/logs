# custom element 속성과 property를 구분하는 법 정리

## 핵심 요약

- 선언형 HTML 입력과 런타임 객체 입력의 책임이 분명해진다.
- attribute 문자열을 boolean 생성자로 바꾸면 'false'도 true가 된다.
- boolean attribute는 hasAttribute로 해석한다.

## 개념 설명

Custom Element의 attribute는 HTML에 직렬화되는 문자열 입력이고 property는 객체, 함수, boolean처럼 DOM 인스턴스가 런타임에 보관하는 JavaScript 값이다.

관찰할 attribute는 `observedAttributes`에 선언해 콜백에서 타입을 변환하고, 복합 값은 property setter로 받아 내부 상태를 갱신하되 양방향 반영 규칙을 명시해야 한다.

## 예시

```ts
class UserBadge extends HTMLElement {
  static observedAttributes = ["compact"];
  #user: User | null = null;
  set user(value: User | null) { this.#user = value; this.render(); }
  attributeChangedCallback() { this.render(); }
  get compact() { return this.hasAttribute("compact"); }
}
customElements.define("user-badge", UserBadge);
```

compact는 존재 여부가 의미인 attribute로, User 객체는 property로 다룬다. 객체를 JSON attribute로 왕복시켜 변경 감지를 흉내 내지 않는다.

## 면접 답변 예시

> property setter에서 렌더링과 검증 시점을 통제할 수 있다. 초기 upgrade 전 설정된 own property가 prototype setter를 가릴 수 있다. connectedCallback에서 upgrade 전 property를 재적용하는 사례를 테스트한다.

## 장점

- boolean attribute를 플랫폼 규칙에 맞게 표현할 수 있다.

## 단점

- property 변경을 무조건 attribute로 반영하면 객체 직렬화 비용이 생긴다.

## 주의사항 / 실무 팁

- 반영이 필요한 원시값만 property와 attribute 사이에서 reflect한다.
