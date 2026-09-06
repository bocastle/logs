# CSS custom media 쿼리 정리

## 핵심 요약

- 반응형 조건을 디자인 토큰처럼 문서화할 수 있다.
- 빌드 도구가 `@custom-media`를 처리하지 않으면 CSS가 적용되지 않을 수 있다.
- `--compact`, `--wide`처럼 조건 이름 규칙을 정한다.

## 개념 설명

CSS custom media는 반복되는 media query 조건에 이름을 붙여 breakpoint와 사용자 환경 조건을 토큰처럼 관리하는 방식이다.

`@custom-media`로 `--narrow` 같은 조건을 선언하고 `@media (--narrow)`로 재사용한다. 현재는 빌드 도구나 브라우저 지원 정책을 확인해야 한다.

## 예시

```css
@custom-media --compact (width <= 48rem);
@media (--compact) {
  .toolbar { flex-wrap: wrap; }
}
```

`@custom-media` 이름으로 좁은 화면 조건을 공유해 breakpoint 숫자 중복을 줄이는 예다.

## 면접 답변 예시

> CSS custom media는 반복되는 media query에 `--compact` 같은 이름을 붙여 breakpoint 의도를 공유하는 방식입니다. 숫자를 여러 파일에 복사하지 않아 변경 지점이 줄지만 target browser가 문법을 직접 처리하는지, build tool이 표준 `@media`로 변환하는지를 먼저 확인해야 합니다. Viewport와 사용자 환경 조건에는 custom media가 잘 맞지만 component 자체 폭에 반응해야 하면 container query가 더 적절할 수 있습니다. 이름에 조건의 목적을 드러내고 build 결과에 유효한 media query가 남는지 테스트하겠습니다.

## 장점

- breakpoint 변경 시 여러 CSS 파일을 덜 수정한다.

## 단점

- 이름이 모호하면 실제 조건을 추적하기 어렵다.

## 주의사항 / 실무 팁

- 빌드 결과에 실제 `@media`가 남는지 확인한다.
