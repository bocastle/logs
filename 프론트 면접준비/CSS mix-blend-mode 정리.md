# CSS mix-blend-mode 정리

## 핵심 요약

- 색 조합에 따라 반응하는 포스터·포토그래픽 효과를 만든다.
- 배경 색이 바뀌면 텍스트 대비가 예상 밖으로 달라진다.
- 텍스트 가독성을 blend 결과에만 의존하지 않는다.

## 개념 설명

CSS `mix-blend-mode`는 요소의 콘텐츠를 직하의 backdrop과 multiply, screen, difference 등 지정한 blending mode로 혼합하는 속성이다.

브라우저는 요소의 source color와 backdrop color를 blend function으로 결합한다. blending backdrop 범위를 제한하려면 조상에 `isolation: isolate`를 설정한다.

## 예시

```css
.poster { isolation: isolate; }
.poster__title { color: white; mix-blend-mode: difference; }
```

poster 안의 title을 직하 배경과 difference로 혼합하되 바깥 page와는 섞이지 않게 격리한다.

## 면접 답변 예시

> `mix-blend-mode`는 element의 rendered color를 뒤의 backdrop과 multiply나 difference 방식으로 섞는 시각 효과입니다. Backdrop이 image load나 theme에 따라 바뀌면 text contrast도 예측하기 어려워 핵심 정보의 가독성을 blend 결과에만 맡기지 않겠습니다. Component root의 `isolation: isolate`로 혼합 범위를 제한하고 fallback color와 outline을 제공합니다. Light·dark, image loading·failure와 forced-colors에서 contrast와 compositing cost를 실제 화면으로 확인합니다.

## 장점

- 별도 이미지 합성 없이 블랜딩을 실행한다.

## 단점

- 중첩 blend와 transparency는 compositing 비용을 키운다.

## 주의사항 / 실무 팁

- 컴포넌트 root에 `isolation: isolate`를 두고 영향 범위를 검수한다.
