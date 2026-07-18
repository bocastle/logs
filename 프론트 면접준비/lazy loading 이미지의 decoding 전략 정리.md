# lazy loading 이미지의 decoding 전략 정리

## 핵심 요약

- `loading="lazy"`는 화면에서 먼 이미지의 요청을 늦추고, `decoding="async"`는 디코딩이 다른 렌더링을 덜 막도록 힌트를 준다.
- 첫 화면의 LCP 후보는 lazy loading에서 제외하고 발견 시점과 우선순위를 별도로 최적화한다.
- `width`와 `height` 또는 안정적인 aspect ratio를 제공해 다운로드 전에도 공간을 확보한다.

## 개념 설명

이미지 로딩에는 네트워크 요청과 디코딩이라는 서로 다른 비용이 있다. Native lazy loading은 브라우저가 화면과의 거리 등을 보고 요청을 늦추고, decoding hint는 받은 이미지의 픽셀 준비를 렌더링과 어떻게 조정할지 제안한다.

둘 다 브라우저에 주는 힌트이며 정확한 시점을 강제하지 않는다. 첫 화면의 대표 이미지를 lazy로 표시하면 LCP 요청 자체가 늦어질 수 있다. 반대로 아래쪽 썸네일은 lazy와 async decoding을 조합해 초기 네트워크 경쟁과 긴 decode 작업을 줄일 수 있다.

## 예시

```html
<img src="thumb.jpg" loading="lazy" decoding="async" width="320" height="180" alt="제품 사진" />
```

아래쪽 썸네일에 적합한 예다. 첫 화면의 핵심 이미지는 `loading="eager"` 기본 동작을 유지하고 필요하면 `fetchpriority`와 preload를 waterfall 근거로 검토한다.

## 면접 답변 예시

> `loading="lazy"`는 화면에서 먼 이미지의 네트워크 요청을 늦추고, `decoding="async"`는 디코딩이 렌더링을 덜 막도록 요청하는 별도 힌트입니다. 첫 화면의 LCP 이미지는 lazy 대상에서 제외하고, 아래쪽 썸네일에 적용하겠습니다. 모든 이미지에는 크기나 aspect ratio를 줘 CLS를 막습니다. Network와 Performance 패널에서 요청 시작, decode 작업과 LCP가 실제로 개선됐는지 확인합니다.

## 장점

- 아래쪽 이미지의 초기 네트워크 경쟁과 디코딩으로 인한 긴 프레임을 줄일 수 있다.
- 브라우저 기본 기능을 사용해 별도 IntersectionObserver 코드 없이 지연 로딩할 수 있다.

## 단점

- LCP 후보까지 lazy 처리하면 첫 화면 이미지 요청이 늦어진다.
- 크기를 지정하지 않으면 이미지가 나타난 뒤 layout shift가 생길 수 있다.

## 주의사항 / 실무 팁

- `width`와 `height` 또는 안정적인 aspect ratio를 지정한다.
- 빠른 스크롤과 많은 이미지가 동시에 viewport에 접근하는 상황에서 요청 burst와 decode 시간을 확인한다.
