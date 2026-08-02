# CSS @property 정리

## 핵심 요약

- `@property`는 custom property의 문법, 상속 여부와 초기값을 등록해 browser가 값을 타입 있게 해석하도록 한다.
- 숫자·길이·색처럼 보간할 수 있는 값의 animation과 invalid value fallback에 유용하다.
- 전역 등록 이름과 `inherits` 의미를 design token 규칙에 맞추고 미지원 환경의 기본 스타일을 유지한다.

## 개념 설명

일반 custom property는 token stream으로 다뤄져 browser가 값의 타입을 미리 알지 못한다. `@property`는 `syntax`, `inherits`, `initial-value` descriptor로 이름을 등록해 `<number>`, `<length>`, `<color>` 같은 타입으로 해석하게 한다.

등록된 값은 시작과 끝 사이를 보간할 수 있어 progress와 theme animation에 사용할 수 있다. Syntax에 맞지 않는 값은 일반 미등록 변수와 다르게 등록된 초기값 또는 상속 규칙에 따라 처리된다. `inherits: false`면 부모 token이 자동으로 내려오지 않는다는 점도 의도해야 한다.

## 예시

```css
@property --progress {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}
```

`--progress`를 숫자로 등록하고 상속하지 않게 한 예다. 실제 property 이름은 전역 registration이므로 library와 application이 충돌하지 않도록 prefix를 정한다.

## 면접 답변 예시

> `@property`는 custom property의 syntax, 상속 여부와 초기값을 browser에 등록하는 at-rule입니다. 숫자나 색처럼 타입을 알면 값 사이를 자연스럽게 보간할 수 있고 잘못된 값의 처리도 명확해집니다. 다만 이름은 문서 전역에 등록되고 `inherits: false`는 부모 값 전달을 막으므로 token 설계와 맞춰야 합니다. 보간 이점이 분명한 변수부터 적용하고 미지원 환경에서도 기본 스타일이 유지되는지 확인합니다.

## 장점

- 숫자·길이와 색 custom property를 타입에 맞게 보간할 수 있다.
- Invalid value의 fallback과 상속 여부를 선언적으로 고정할 수 있다.

## 단점

- Syntax가 실제 사용 값과 다르면 초기값으로 돌아가 의도한 style이 사라질 수 있다.
- 전역 property 이름과 token prefix가 겹치면 다른 component의 등록과 충돌한다.

## 주의사항 / 실무 팁

- 등록 전후의 computed value와 미지원 browser의 fallback을 함께 확인한다.
- Design token prefix, `inherits` 규칙과 허용 단위를 문서화한다.
