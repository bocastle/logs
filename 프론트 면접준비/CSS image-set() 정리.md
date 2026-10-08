# CSS image-set() 정리

## 핵심 요약

- CSS background 이미지도 DPR과 포맷별 최적화를 적용할 수 있다.
- LCP 후보를 CSS background로 숨기면 preload와 우선순위 관리가 어려울 수 있다.
- 핵심 콘텐츠 이미지는 가능하면 `img`와 `srcset`을 먼저 검토한다.

## 개념 설명

`image-set()`은 CSS 배경 이미지에서 해상도, 포맷, 타입 후보를 제공해 브라우저가 적절한 이미지를 고르게 하는 함수다.

각 후보에 `1x`, `2x` 같은 resolution descriptor와 `type()` 힌트를 붙이면 DPR과 지원 포맷에 맞는 asset 선택이 가능하다.

## 예시

```css
.hero {
  background-image: image-set(
    url("hero.avif") type("image/avif") 1x,
    url("hero@2x.avif") type("image/avif") 2x,
    url("hero.jpg") type("image/jpeg") 1x
  );
}
```

`image-set()`에 `type()`과 resolution 후보를 넣어 AVIF 지원 여부와 DPR에 따라 배경 이미지를 고르게 하는 예다.

## 면접 답변 예시

> CSS `image-set()`은 background image에 format과 resolution 후보를 주고 browser가 지원 형식과 DPR에 맞는 asset을 고르게 합니다. 고해상도 화면의 선명도를 높일 수 있지만 descriptor와 실제 pixel 크기가 맞지 않으면 필요 이상으로 큰 file을 받을 수 있습니다. DevTools network에서 format·DPR별 실제 선택과 transfer byte를 확인하겠습니다. Hero처럼 LCP에 중요한 content image는 discovery와 priority, alt text를 다루기 쉬운 `img`와 `srcset`을 먼저 검토합니다.

## 장점

- 지원하지 않는 이미지 포맷 fallback을 선언 안에 둘 수 있다.

## 단점

- 후보 descriptor가 실제 파일 크기와 맞지 않으면 과한 이미지를 받을 수 있다.

## 주의사항 / 실무 팁

- background가 필요할 때만 `image-set()`을 쓰고 byte 차이를 측정한다.
