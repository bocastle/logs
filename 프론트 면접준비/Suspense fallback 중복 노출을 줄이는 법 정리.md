# Suspense fallback 중복 노출을 줄이는 법 정리

## 핵심 요약

- Suspense fallback을 layout 경계 기준으로 정리하면 같은 로딩 상태가 화면 여러 곳에 반복되지 않아 사용자가 기다리는 위치를 빨리 이해한다.
- Suspense boundary를 너무 크게 합치면 작은 위젯 하나의 지연이 전체 section fallback으로 번져 초기 표시가 늦어진다.
- 중복 fallback을 줄일 때는 실제 화면 캡처로 어떤 layout 조각이 반복 노출되는지 먼저 표시하고 boundary 후보를 정한다.

## 개념 설명

Suspense fallback 중복 노출을 줄이는 작업은 같은 지연을 상위 shell과 하위 영역의 중첩 fallback으로 반복해서 보여주지 않도록 boundary와 layout 책임을 조정하는 설계다.

상위 Suspense는 페이지 골격이나 route shell만 맡기고, 데이터가 비슷한 속도로 준비되는 하위 컴포넌트는 같은 boundary로 묶거나 이미 보이는 layout 안에서 부분 skeleton만 교체한다.

## 예시

```tsx
<Suspense fallback={<PageShellSkeleton />}>
  <ProfileLayout>
    <Suspense fallback={<ActivityListSkeleton />}>
      <ActivityList />
    </Suspense>
  </ProfileLayout>
</Suspense>
```

layout은 유지하고 실제로 늦는 목록만 fallback으로 바꾸면 헤더, 탭, 카드 골격이 여러 번 깜빡이는 중복 노출을 줄인다.

## 면접 답변 예시

> 상위 shell을 안정적으로 남기면 route 전환 중에도 탐색 맥락과 기존 콘텐츠 높이가 유지되어 누적 레이아웃 이동을 줄인다. 서버 스트리밍 boundary와 클라이언트 데이터 cache가 어긋나면 중복 skeleton보다 더 큰 hydration 지연이 생길 수 있다. 개선 뒤에는 느린 3G, cache miss, 재시도 흐름에서 skeleton 개수와 layout shift가 줄었는지 함께 확인한다.

## 장점

- 데이터 의존성이 같은 위젯을 하나의 boundary에 묶으면 skeleton cascade가 줄고 reveal 순서를 제품 흐름에 맞출 수 있다.

## 단점

- fallback을 모두 제거하면 느린 네트워크에서 빈 영역만 남아 사용자가 진행 중인지 실패인지 구분하기 어렵다.

## 주의사항 / 실무 팁

- route shell, 핵심 콘텐츠, 보조 패널처럼 사용자가 의미를 다르게 읽는 영역별로 Suspense fallback 크기와 문맥을 다르게 둔다.
