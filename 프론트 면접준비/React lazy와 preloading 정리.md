# React lazy와 preloading 정리

## 핵심 요약

- React.lazy로 드물게 쓰는 화면 chunk를 분리하면 초기 경로가 내려받고 파싱할 JavaScript 양이 줄어든다.
- 사용 가능성이 낮은 모든 route를 즉시 preload하면 대역폭 경쟁으로 현재 화면의 핵심 리소스가 늦어진다.
- loader 함수를 모듈 범위에 두고 사용자 intent와 네트워크 상태를 기준으로 같은 함수를 미리 호출한다.

## 개념 설명

`React.lazy`는 컴포넌트 코드를 동적으로 불러오고 Suspense fallback으로 로딩 상태를 표현한다.

사용자가 곧 방문할 경로나 열 가능성이 높은 패널은 import를 미리 시작해 첫 렌더 지연을 줄일 수 있다.

## 예시

```tsx
const Settings = lazy(() => import("./Settings"));
button.onPointerEnter = () => void import("./Settings");
```

hover 시점에 chunk 요청을 시작하면 실제 클릭 뒤 Suspense fallback 시간이 짧아진다.

## 면접 답변 예시

> 동일한 import Promise를 재사용하면 preload와 실제 lazy 렌더가 하나의 네트워크 요청을 공유한다. 배포 뒤 오래된 HTML이 사라진 chunk를 요청하면 동적 import가 reject되어 fallback만으로 복구되지 않는다. 번들 분석과 실제 route 전환 trace로 초기 JS 감소량과 preload 적중률을 함께 측정한다.

## 장점

- hover나 viewport 진입 때 preloading을 시작하면 클릭 후 Suspense fallback이 보이는 시간을 단축한다.

## 단점

- 컴포넌트 함수 안에서 React.lazy를 새로 만들면 렌더마다 타입 정체성이 바뀌어 자식 state가 초기화된다.

## 주의사항 / 실무 팁

- chunk 오류는 Error Boundary에서 감지해 한 번의 새로고침 또는 최신 asset 재시도 경로를 제공한다.
