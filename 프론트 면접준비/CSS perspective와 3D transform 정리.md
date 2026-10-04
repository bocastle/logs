# CSS perspective와 3D transform 정리

## 핵심 요약

- 여러 자식이 같은 관찰점을 공유하는 3D scene을 만든다.
- perspective 거리가 짧을수록 원근 효과가 과격해져 변형이 어지러울 수 있다.
- 여러 object의 공유 scene은 부모 `perspective`, 개별 object 효과는 `perspective()`를 기준으로 고른다.

## 개념 설명

CSS `perspective` property는 요소의 3D-transformed 자식들에 공유 원근 투영을 적용하고, `perspective()` transform function은 함수가 들어간 요소 자신의 현재 변환 행렬에 원근 투영을 포함한다.

부모의 `perspective`/`perspective-origin`은 자식이 공유하는 관찰 거리와 소실점을 정한다. `transform: perspective(800px) rotateY(35deg)`의 `perspective()`는 해당 변환 함수 목록의 순서로 계산되며, `transform-style: preserve-3d`는 하위 요소의 3D 공간을 유지한다.

## 예시

```css
.scene { perspective: 800px; perspective-origin: 50% 40%; }
.card { transform-style: preserve-3d; transform: rotateY(35deg); }
.single { transform: perspective(800px) rotateY(35deg); }
```

scene의 자식 card들은 공유 소실점을 사용하지만 `.single`은 자신의 transform list에만 `perspective()`를 적용한다.

## 면접 답변 예시

> `perspective-origin`으로 소실점을 scene 레이아웃에 맞춘다. overflow, opacity, filter 등 grouping property는 descendant 3D context를 flatten할 수 있다. `prefers-reduced-motion`에서 큰 3D 회전을 정적 배치로 대체한다.

## 장점

- Z 위치가 가까운 요소는 크게, 먼 요소는 작게 표현해 깊이감을 만든다.

## 단점

- 변환 함수 목록에서 `perspective()` 순서가 바뀌면 행렬 곱 결과도 바뀐다.

## 주의사항 / 실무 팁

- 변환 함수 순서와 computed `matrix3d()`를 DevTools에서 확인한다.
