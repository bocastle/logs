# PerformanceObserver로 long task를 추적하는 법 정리

## 핵심 요약

- Long task entry는 메인 스레드를 50ms 넘게 점유한 구간을 실제 사용자 환경에서 찾게 한다.
- Entry만으로 정확한 JavaScript stack을 알 수 없으므로 route·interaction 시각과 배포 version을 함께 기록한다.
- RUM은 sampling과 duration 구간을 정하고 대표 사례를 DevTools profile로 다시 재현한다.

## 개념 설명

`PerformanceObserver`는 performance entry를 비동기로 받아 처리하는 API다. `longtask` entry는 main thread의 하나의 task가 50ms를 넘긴 구간을 알려 주며 hydration, 큰 script 실행과 복잡한 rendering 회귀를 찾는 데 사용할 수 있다.

Long task가 있다고 모든 사용자 입력이 느렸다는 뜻은 아니다. 발생 시각을 interaction과 INP, route 전환에 연결해야 사용자 영향을 판단할 수 있다. Observer entry는 stack trace를 제공하지 않으므로 실제 원인 함수는 재현 가능한 사례의 Performance profile로 확인한다.

## 예시

```js
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) report(entry.duration);
}).observe({ type: "longtask", buffered: true });
```

Duration과 start time을 현재 route, 최근 interaction과 안전한 component marker에 연결한다. Background tab이나 bot traffic을 섞지 않고 모든 entry를 전송하지 않도록 sampling한다.

## 면접 답변 예시

> `PerformanceObserver`의 long task entry로 실제 사용자 기기에서 main thread를 50ms 넘게 점유한 구간을 찾을 수 있습니다. 다만 entry만으로 원인 함수와 입력 지연을 확정할 수 없으므로 route, 최근 interaction과 INP를 같은 시간축으로 보겠습니다. RUM 데이터는 sampling하고 background tab을 구분합니다. 회귀가 보이면 같은 조건을 DevTools Performance profile로 재현해 stack과 rendering 비용을 찾습니다.

## 장점

- INP와 route 전환 악화를 main thread의 긴 실행 구간과 시간상 연결할 수 있다.
- 운영 기기 성능 분포에서 배포 후 회귀를 빠르게 찾을 수 있다.

## 단점

- Entry 자체에는 정확한 JavaScript stack이 없어 추가 재현이 필요하다.
- 모든 entry를 전송하면 RUM 비용과 사용자 telemetry 양이 커진다.

## 주의사항 / 실무 팁

- Sampling 비율과 duration bucket을 정하고 background visibility를 함께 기록한다.
- 대표 route와 저사양 기기 조건을 DevTools profile로 재현해 원인 함수를 확인한다.
