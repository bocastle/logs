# Long Animation Frames API 정리

## 핵심 요약

- 스크롤과 애니메이션 jank를 프레임 단위로 볼 수 있다.
- 지원하지 않는 브라우저에서는 데이터가 빠질 수 있다.
- LoAF는 INP, longtask와 함께 수집한다.

## 개념 설명

Long Animation Frames API는 렌더링 프레임이 길어진 원인과 시간을 `long-animation-frame` entry로 관찰하는 성능 API다.

LoAF entry는 긴 프레임의 duration, script work, `blockingDuration` 같은 정보를 제공해 단순 long task보다 프레임 단위 jank를 보기 좋다.

## 예시

```js
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) report(entry.duration, entry.blockingDuration);
}).observe({ type: "long-animation-frame", buffered: true });
```

`long-animation-frame` entry의 `blockingDuration`을 기록해 애니메이션이나 입력 중 프레임이 막히는 구간을 찾는다.

## 면접 답변 예시

> Long Animation Frames API는 browser rendering frame이 길어진 구간을 `long-animation-frame` entry로 관찰하는 기능입니다. `duration`과 `blockingDuration`, 관련 script 정보를 보면 long task만 볼 때보다 interaction 중 jank가 생긴 frame을 좁히기 좋습니다. 다만 RUM entry만으로 원인 코드를 확정하기는 어려워 route와 interaction 시점을 함께 기록하고 대표 환경에서 trace를 재현하겠습니다. 지원하지 않는 browser의 데이터 공백과 전송량을 고려해 INP·long task와 함께 sampling해서 봅니다.

## 장점

- 긴 script와 rendering 비용을 INP 악화 후보와 연결한다.

## 단점

- entry 수가 많아 샘플링 없이 전송하면 비용이 커진다.

## 주의사항 / 실무 팁

- duration과 `blockingDuration` threshold를 분리한다.
