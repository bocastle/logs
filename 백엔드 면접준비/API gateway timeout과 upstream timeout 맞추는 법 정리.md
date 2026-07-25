# API gateway timeout과 upstream timeout 맞추는 법 정리

## 핵심 요약

- 바깥 gateway timeout보다 안쪽 upstream timeout을 짧게 둬 오류 변환과 정리에 필요한 시간을 남긴다.
- 하나의 값을 모든 route에 적용하지 않고 SLO, 정상 tail latency와 재시도 횟수에서 요청 budget을 정한다.
- Gateway가 응답을 포기한 뒤에도 DB와 HTTP 작업이 계속되지 않도록 deadline과 취소를 하위 계층까지 전파한다.

## 개념 설명

여러 계층의 timeout이 같거나 안쪽이 더 길면 gateway가 client 연결을 끊은 뒤에도 upstream은 결과를 만들기 위해 자원을 사용한다. 한 요청의 전체 지연 예산을 network, application, dependency와 오류 처리 구간으로 나눠 안쪽부터 먼저 종료되게 해야 한다.

Route SLO와 실제 p95·p99를 기준으로 gateway 상한을 정하고 network·serialization·fallback 여유를 뺀 값을 upstream에 준다. 호출이 여러 단계라면 고정 timeout을 새로 시작하지 말고 현재 남은 deadline을 전달한다. 취소 signal이 DB driver와 downstream client까지 실제로 적용되는지도 확인한다.

## 예시

```text
route SLO budget: 3000ms
gateway timeout: 2800ms
upstream timeout: 2400ms
reserve: 400ms for network, mapping, fallback
```

예시에서는 upstream이 2.4초에 끝나 gateway가 0.4초 안에 오류를 변환할 수 있다. 이미 여러 번 재시도한다면 각 attempt의 timeout과 backoff도 같은 2.8초 안에 들어와야 한다.

## 면접 답변 예시

> Gateway timeout은 요청 전체의 바깥 상한이고 upstream timeout은 그 안에서 더 짧아야 합니다. Route SLO와 tail latency로 예산을 정하고 network, 응답 변환과 fallback 시간을 남긴 뒤 upstream deadline을 계산하겠습니다. 요청이 여러 서비스를 지나면 남은 시간을 전달하고 client 취소가 DB와 HTTP 호출까지 전파되게 합니다. 느린 dependency와 재시도를 포함한 통합 테스트에서 gateway가 끊기기 전에 명확한 오류를 만들 수 있는지 확인합니다.

## 장점

- Client 응답이 끊긴 뒤에도 남는 upstream·DB 작업을 줄인다.
- 계층별 timeout 원인과 남은 budget을 지표로 구분하기 쉬워진다.

## 단점

- 모든 route에 하나의 budget을 사용하면 정상적인 장기 작업이 잘리거나 짧은 API가 너무 오래 대기한다.
- 취소를 지원하지 않는 driver가 있으면 timeout 뒤에도 실제 작업과 lock이 남을 수 있다.

## 주의사항 / 실무 팁

- Gateway timeout, upstream timeout, deadline 잔여 시간과 cancel count를 같은 대시보드에 둔다.
- 느린 dependency, retry와 fallback을 포함한 경계값 테스트로 실제 종료 순서를 검증한다.
