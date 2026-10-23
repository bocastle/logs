# React Virtual DOM 없는 프레임워크 비교 정리

## 핵심 요약

- Virtual DOM과 fine-grained 반응성을 비교하면 Fiber scheduling과 직접 구독의 업데이트 단위 차이를 이해할 수 있다.
- Virtual DOM이 있다는 이유만으로 항상 느리다고 결론 내리면 compiler 최적화와 batching 및 실제 DOM 비용을 무시한다.
- 같은 데이터 크기, 상호작용, production build에서 scripting, layout, memory를 브라우저 trace로 비교한다.

## 개념 설명

React Virtual DOM 비교는 변경을 가상 트리 diff로 처리하는 방식과 컴파일 타임 또는 fine-grained signal 방식의 차이를 보는 주제다.

React는 render 결과를 Fiber tree로 비교하고, 다른 프레임워크는 의존성 단위 업데이트나 컴파일된 DOM 명령으로 비용 모델을 바꾼다.

## 예시

```text
React: state change -> render -> Fiber diff -> commit
Signal 기반: signal change -> dependent effect -> DOM update
```

프레임워크 우열보다 업데이트 단위, 디버깅 방식, 생태계 제약을 함께 비교해야 한다.

## 면접 답변 예시

> React의 생태계와 concurrent 기능을 성능 숫자와 함께 평가해 프레임워크 교체의 전체 비용을 볼 수 있다. 한 프레임워크에 익숙한 구현을 다른 쪽의 비관용 코드와 비교하면 렌더 모델보다 작성 품질을 측정하게 된다. 성능 외에 bundle, hydration, 팀 학습, 라이브러리 호환성을 포함한 결정 기록을 남긴다.

## 장점

- 동일한 제품 요구를 여러 모델로 측정해 초기 실행 비용과 반복 갱신 비용 중 중요한 축을 선택할 수 있다.

## 단점

- Fiber와 signal 기반 런타임의 서로 다른 개발 도구, SSR, 접근성 지원을 제외한 microbenchmark는 의사결정을 왜곡한다.

## 주의사항 / 실무 팁

- 초기 mount, 단일 cell 갱신, 큰 목록 교체처럼 update granularity가 다른 시나리오를 분리해 측정한다.
