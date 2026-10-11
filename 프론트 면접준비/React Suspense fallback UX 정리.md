# React Suspense fallback UX 정리

## 핵심 요약

- 늦는 Article 영역만 fallback으로 바꾸면 사용자는 헤더와 탐색 위치를 유지한 채 기다릴 수 있다.
- 매우 빠른 요청에도 skeleton을 즉시 보이면 짧은 fallback flash가 오히려 화면을 불안정하게 만든다.
- skeleton의 width와 height를 실제 콘텐츠와 맞추고 애니메이션은 prefers-reduced-motion을 존중한다.

## 개념 설명

Suspense fallback UX는 데이터나 코드가 준비될 때까지 사용자가 보게 되는 대기 화면의 크기와 위치를 설계하는 일이다.

경계를 너무 크게 잡으면 전체 화면이 사라지고, 너무 작게 잡으면 skeleton이 여러 곳에서 흔들려 맥락을 잃는다.

## 예시

```tsx
<Suspense fallback={<ArticleSkeleton />}>
  <Article />
</Suspense>
```

사용자가 기다리는 영역만 skeleton으로 바꾸면 기존 탐색 맥락이 유지된다.

## 면접 답변 예시

> 경계를 데이터 준비 단위에 맞추면 빠른 영역부터 점진적으로 보여 체감 초기 응답을 앞당긴다. 페이지 전체를 한 Suspense로 감싸면 작은 위젯 지연 때문에 이미 보이던 탐색 UI까지 사라진다. cache hit, 빠른 응답, 느린 응답에서 fallback 노출 시간과 layout shift를 실제 브라우저 trace로 확인한다.

## 장점

- 실제 콘텐츠와 같은 크기의 skeleton은 reveal 전후 누적 레이아웃 이동을 줄인다.

## 단점

- 장식용 회색 box만 나열하고 aria-busy 상태를 알리지 않으면 보조기술 사용자는 대기 중임을 알기 어렵다.

## 주의사항 / 실무 팁

- 기존 콘텐츠를 유지해야 하는 갱신에는 transition을 사용해 초기 load fallback과 재조회 표시를 구분한다.
