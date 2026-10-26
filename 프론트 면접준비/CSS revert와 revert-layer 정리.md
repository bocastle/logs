# CSS revert와 revert-layer 정리

## 핵심 요약

- 과한 reset이나 component override를 국소적으로 되돌릴 수 있다.
- `revert`와 `revert-layer` 차이를 모르면 예상보다 많은 스타일이 사라진다.
- 되돌릴 범위를 속성 단위로 먼저 제한한다.

## 개념 설명

`revert`와 `revert-layer`는 현재 cascade에서 적용된 값을 브라우저 기본값이나 이전 layer 값으로 되돌리는 CSS 키워드다.

`revert`는 user-agent 또는 이전 origin 기준으로 되돌리고, `revert-layer`는 현재 cascade layer의 선언만 걷어내 layer 설계 안에서 reset을 제어한다.

## 예시

```css
@layer components {
  .prose button { all: revert-layer; }
}
```

`revert-layer`로 components layer의 과한 버튼 스타일을 걷어내고 다른 cascade 규칙은 유지하는 예다.

## 면접 답변 예시

> `revert`는 현재 author origin의 값을 이전 origin이나 UA 기본 흐름으로 되돌리고, `revert-layer`는 현재 cascade layer의 선언만 걷어내 이전 layer 결과를 드러냅니다. Component override를 국소적으로 취소할 때는 전체 cascade를 되돌리기보다 필요한 property나 layer 범위를 먼저 선택하겠습니다. `all: revert-layer`는 편리하지만 focus와 disabled 같은 상태 style까지 예상과 다르게 바꿀 수 있습니다. Layer 순서를 문서화하고 normal, focus, forced-colors 상태의 computed style을 확인합니다.

## 장점

- cascade layer 설계에서 탈출구를 명확히 둔다.

## 단점

- layer 순서가 바뀌면 되돌림 결과도 달라진다.

## 주의사항 / 실무 팁

- layer 순서를 문서화한 뒤 사용한다.
