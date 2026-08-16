# Resource Timing API 정리

## 핵심 요약

- Resource Timing API는 페이지가 불러온 이미지, 스크립트, CSS 같은 개별 리소스의 네트워크 구간을 측정한다.
- `PerformanceResourceTiming`의 `responseStart`, `duration`, `transferSize`를 함께 보면 서버 대기와 전송 비용을 나눠 볼 수 있다.
- 교차 출처 리소스의 상세 타이밍은 서버가 `Timing-Allow-Origin`을 허용해야 확인할 수 있다.

## 개념 설명

브라우저는 리소스마다 DNS, 연결, 요청, 응답 시점을 performance entry buffer에 기록한다. 캐시 적중 여부는 `transferSize`, `encodedBodySize`, protocol 정보를 함께 보고 추정한다.

수집 결과는 실제 사용자 환경의 병목을 찾는 데 유용하지만, 버퍼 크기와 샘플링을 관리하지 않으면 관측 코드 자체가 비용이 될 수 있다.

## 예시

```ts
const resources = performance.getEntriesByType("resource")
  as PerformanceResourceTiming[];

for (const entry of resources) {
  report({
    name: entry.name,
    ttfb: entry.responseStart - entry.requestStart,
    duration: entry.duration,
    transferSize: entry.transferSize,
  });
}
```

TTFB와 전체 duration을 나누고 `transferSize`를 같이 보내면 서버 대기, 큰 payload, 캐시 미적중 중 어느 쪽이 문제인지 좁히기 쉽다.

## 면접 답변 예시

> Resource Timing API는 image, script와 CSS 같은 개별 resource의 network timing을 `PerformanceResourceTiming`으로 확인하는 기능입니다. `responseStart - requestStart`, 전체 `duration`과 `transferSize`를 함께 보면 server 대기, 전송량과 cache 효과를 나눠 볼 수 있습니다. Cross-origin resource의 상세 timing은 server가 `Timing-Allow-Origin`을 허용해야 합니다. RUM에서는 URL을 정규화하고 sampling해 LCP와 long task 같은 사용자 지표와 함께 분석하겠습니다.

## 장점

- 실제 사용자 환경에서 느린 리소스를 URL 단위로 찾을 수 있다.
- 서버 대기와 다운로드 시간을 나눠 최적화 우선순위를 정할 수 있다.
- 캐시와 전송 프로토콜의 효과를 같은 데이터에서 확인할 수 있다.

## 단점

- 교차 출처 서버 설정이 없으면 상세 구간이 제한된다.
- 모든 엔트리를 수집하면 전송량과 분석 비용이 커진다.
- 단일 측정값만으로 사용자 체감 성능 전체를 설명할 수는 없다.

## 주의사항 / 실무 팁

- RUM에서는 사용자·세션별 샘플링 비율을 정한다.
- 리소스 URL의 쿼리 문자열에 민감정보가 섞이지 않게 정규화한다.
- LCP와 long task 같은 사용자 지표와 함께 분석한다.
