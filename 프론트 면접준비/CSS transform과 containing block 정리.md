# CSS transform과 containing block 정리

## 핵심 요약

- transform 뒤 위치 계산이 변하는 원인을 설명한다.
- 성능 최적화라고 `translateZ(0)`를 남발하면 fixed UI 기준이 바뀐다.
- fixed 위치 버그는 조상의 `transform`, `filter`, `backdrop-filter`, `perspective`를 확인하고 `isolation`은 containing block이 아닌 stacking context 격리임을 구분한다.

## 개념 설명

CSS `transform`이 `none`이 아닌 요소는 시각적 변환 뿐 아니라 positioned descendant의 containing block과 자신의 stacking context를 만들 수 있다.

transformed element의 padding box가 `position: absolute`와 `position: fixed` descendant의 containing block 기준이 될 수 있어 fixed 요소가 viewport 대신 변환된 조상을 따라 scroll될 수 있다. 3D transform 함수 자체가 원근감을 만드는 것은 아니며 부모 `perspective` 속성 또는 `perspective()`가 필요하다.

## 예시

```css
.shell { transform: translateZ(0); }
.shell .toast { position: fixed; inset: 1rem; }
```

`.shell`의 transform 때문에 toast의 fixed containing block이 viewport가 아닌 shell이 될 수 있는 예다.

## 면접 답변 예시

> `transform`이 `none`이 아닌 element는 시각적 변환뿐 아니라 stacking context와 positioned descendant의 containing block을 만들 수 있습니다. 그래서 transformed ancestor 안의 `position: fixed`가 viewport가 아니라 그 ancestor를 기준으로 움직이는 문제가 생길 수 있습니다. Transform은 normal flow의 점유 공간을 다시 계산하지 않으므로 보이는 위치와 layout 공간도 다를 수 있습니다. 단순 compositing 기대만으로 `translateZ(0)`을 남발하지 않고 fixed bug에서는 조상의 transform과 filter 계열을 확인하며 overlay는 필요하면 subtree 밖 portal에 둡니다.

## 장점

- 변환된 컴포넌트 안의 absolute 요소를 지역 기준으로 배치한다.

## 단점

- stacking context가 생겨 바깥 overlay 위로 올라가지 못할 수 있다.

## 주의사항 / 실무 팁

- overlay를 transformed subtree 밖 portal에 두는 설계를 검토한다.
