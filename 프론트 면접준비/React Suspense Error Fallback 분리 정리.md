# React Suspense Error Fallback 분리 정리

## 핵심 요약

- Suspense는 pending skeleton을, Error Boundary는 실패와 재시도를 맡아 사용자가 현재 상태를 구분할 수 있다.
- 하나의 fallback을 대기와 실패에 재사용하면 자동 진행 중인지 사용자 조치가 필요한지 알 수 없다.
- Error Boundary 바깥이나 안쪽에 Suspense를 배치할 때 어느 영역까지 오류 UI로 교체할지 화면 단위로 결정한다.

## 개념 설명

Suspense fallback은 대기 상태를, Error Boundary fallback은 실패 상태를 보여주므로 두 경계를 분리해 설계해야 한다.

Promise pending은 Suspense가 받고 reject나 render 오류는 Error Boundary가 받기 때문에 복구 버튼과 skeleton의 책임이 다르다.

## 예시

```tsx
<ErrorBoundary fallback={<RetryPanel />}>
  <Suspense fallback={<UserSkeleton />}>
    <UserPanel />
  </Suspense>
</ErrorBoundary>
```

로딩 skeleton과 재시도 UI가 섞이지 않아 사용자가 현재 상태를 더 정확히 이해한다.

## 면접 답변 예시

> 로딩과 오류 telemetry를 서로 다른 경계에서 수집해 지연 문제와 실제 실패율을 분리해 볼 수 있다. 실패한 Promise를 같은 캐시 key로 계속 읽으면 reset 버튼만 눌러도 즉시 같은 오류가 다시 발생한다. pending, reject, retry 성공을 각각 강제한 통합 테스트에서 shell과 두 fallback의 노출 순서를 확인한다.

## 장점

- 오류 경계를 작은 데이터 영역에 두면 한 위젯의 reject가 이미 준비된 페이지 shell을 가리지 않는다.

## 단점

- Error Boundary는 event handler와 임의 비동기 callback의 throw를 잡지 않으므로 모든 오류 처리 수단이 되지 않는다.

## 주의사항 / 실무 팁

- 재시도 동작은 오류 state뿐 아니라 실패한 resource cache를 무효화하고 새 Promise를 시작하게 만든다.
