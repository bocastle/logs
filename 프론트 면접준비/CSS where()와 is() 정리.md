# CSS where()와 is() 정리

## 핵심 요약

- 선택자 중복을 줄인다.
- `:is()` 인자의 ID 하나가 전체 specificity를 높일 수 있다.
- 기본 컴포넌트 스타일에는 `:where()`를 우선 검토한다.

## 개념 설명

CSS `:where()`와 `:is()`는 여러 selector 후보를 하나의 functional pseudo-class로 묶지만 specificity 계산이 다르다.

`:where()`는 인자와 관계없이 specificity가 0이고, `:is()`는 인자 selector list에서 가장 높은 specificity를 사용한다. 둘 모두 forgiving selector list를 사용한다.

## 예시

```css
:where(.prose, .editor) a { color: var(--link); }
:is(#app, .preview) .notice { font-weight: 700; }
```

라이브러리 기본 링크는 낮은 specificity로 두고 `:is()` 규칙은 인자의 가장 높은 specificity를 가진다.

## 면접 답변 예시

> `:where()`와 `:is()`는 여러 selector 후보를 묶지만 specificity 계산이 다릅니다. `:where()`는 인자와 관계없이 specificity가 0이라 library 기본 style처럼 쉽게 override해야 하는 규칙에 적합합니다. `:is()`는 인자 중 가장 높은 specificity를 사용하므로 ID가 하나 섞이면 전체 규칙이 예상보다 강해질 수 있습니다. 중첩 selector는 DevTools의 matched rule과 computed specificity로 실제 cascade를 확인하겠습니다.

## 장점

- `:where()`로 쉬운 override를 설계한다.

## 단점

- `:where()`를 권한 override에 쓰면 예상보다 쉽게 덮어쓰여진다.

## 주의사항 / 실무 팁

- `:is()` 인자에 ID를 넣을 때 computed specificity를 확인한다.
