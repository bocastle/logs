# CSS logical shorthand 정리

## 핵심 요약

- LTR·RTL 레이아웃에 같은 component CSS를 재사용한다.
- logical과 physical property를 같은 cascade 우선순위에 섞으면 최종 값을 읽기 어렵다.
- 새 component의 spacing·size·inset은 logical property를 기본으로 한다.

## 개념 설명

CSS logical shorthand는 `margin-inline`, `padding-block`, `inset-inline`, `border-block`처럼 left·right·top·bottom 대신 writing mode의 inline·block 축을 기준으로 여러 물리 속성을 묶는 CSS 속성이다.

inline shorthand의 첫·두 값은 inline-start·inline-end, block shorthand의 첫·두 값은 block-start·block-end에 매핑된다. 결과는 `direction`과 `writing-mode`에 따라 물리 방향이 바뀐다.

## 예시

```css
.card {
  margin-block: 1rem 2rem;
  padding-inline: 1.25rem;
  border-inline-start: 4px solid var(--accent);
}
```

card 여백과 강조 선을 logical axis로 지정해 LTR·RTL·vertical writing에 따라 매핑되게 한다.

## 면접 답변 예시

> CSS logical shorthand는 left·right가 아니라 writing mode의 inline·block 축으로 여백, border와 inset을 표현하는 방법입니다. Component spacing에는 `margin-inline`과 `padding-block`을 기본으로 써서 LTR·RTL과 세로 쓰기에서도 같은 의도를 유지하겠습니다. 지도 방향이나 media control처럼 물리적 방향 자체가 의미인 경우만 physical property를 사용하고, 두 체계를 같은 cascade에 섞을 때는 override 순서를 명확히 합니다. LTR, RTL과 `vertical-rl` 화면을 회귀 캡처해 start·end 매핑과 overflow를 확인합니다.

## 장점

- vertical writing mode에서 여백 의도를 유지한다.

## 단점

- 시간 축·지도 방향처럼 물리 left/right이 의미인 UI에는 적합하지 않을 수 있다.

## 주의사항 / 실무 팁

- 화살표·메디아 컨트롤 같은 물리 방향 예외를 문서화한다.
