# PerformanceObserver 정리

## 핵심 요약

- RUM 수집을 브라우저 성능 entry와 직접 연결한다.
- 지원하지 않는 entry type은 조용히 빠질 수 있다.
- `PerformanceObserver.supportedEntryTypes`를 확인한다.

## 개념 설명

`PerformanceObserver`는 navigation, resource, paint, layout-shift 같은 성능 entry가 생길 때 콜백으로 받는 브라우저 API다.

`observe({ type, buffered: true })`를 쓰면 이미 발생한 entry까지 받아 초기 로딩 지표를 놓치지 않는다. 지원 entry type은 브라우저마다 확인해야 한다.

## 예시

```js
const observer = new PerformanceObserver((list) => {
  send(list.getEntries().map((entry) => ({ name: entry.name, duration: entry.duration })));
});
observer.observe({ type: "resource", buffered: true });
```

리소스 entry를 모아 전송하는 기본 구조다. 관찰 대상 type을 명확히 고르지 않으면 데이터는 많아도 답을 주지 못한다.

## 면접 답변 예시

> `PerformanceObserver`는 navigation, resource, paint 같은 performance entry를 비동기로 수집하는 API입니다. 초기 지표가 필요하면 지원 여부를 확인한 뒤 `buffered: true`를 사용하고, 관찰 목적에 맞는 entry type만 선택합니다. Resource name에는 query나 민감한 URL이 섞일 수 있어 전송 전에 정규화합니다. 수집량과 사용자 비용을 줄이기 위해 sampling과 buffer 처리 정책도 함께 둡니다.

## 장점

- 초기 로딩 이후 생기는 지표도 지속적으로 받을 수 있다.

## 단점

- 수집량을 제어하지 않으면 분석 비용이 커진다.

## 주의사항 / 실무 팁

- `buffered`가 필요한 초기 지표를 따로 정한다.
