# HTTP-3가 프론트 리소스 로딩에 주는 영향 정리

## 핵심 요약

- HTTP/3는 QUIC 위에서 stream별 손실 복구와 빠른 연결 수립을 제공하지만 모든 환경에서 자동으로 빨라지는 것은 아니다.
- CDN·network가 실제로 h3를 사용했는지 `nextHopProtocol`과 server log로 확인한다.
- TTFB, resource duration과 LCP를 h2 fallback 사용자와 같은 조건에서 비교한다.

## 개념 설명

HTTP/3는 TCP 대신 UDP 기반 QUIC 위에서 HTTP semantics를 전달한다. QUIC은 TLS를 통합하고 여러 stream을 독립적으로 복구해 한 stream의 packet loss가 다른 stream까지 transport 수준에서 기다리게 하는 현상을 줄인다.

재방문 연결에서는 0-RTT 가능성도 있지만 replay 위험 때문에 모든 요청에 안전하게 사용할 수 있는 기능은 아니다. Frontend 효과는 사용자의 손실률, CDN 위치, connection reuse와 resource 구성에 따라 달라진다. 일부 network가 UDP를 제한하면 h2로 fallback할 수 있다.

## 예시

```text
비교 지표: protocol=h3, resource duration, TTFB, LCP
대상: JS chunk, CSS, LCP image
```

Resource Timing의 `nextHopProtocol`을 이용해 실제 h3와 h2 표본을 나누고 같은 핵심 resource의 TTFB와 duration을 본다. 단순 protocol 사용률 상승을 LCP 개선으로 해석하지 않는다.

## 면접 답변 예시

> HTTP/3는 QUIC 위에서 stream별 손실 복구와 빠른 연결 수립을 제공하는 HTTP 전송 방식입니다. 손실이 있는 mobile network에서는 여러 resource의 대기 시간을 줄일 수 있지만 CDN과 network 조건에 따라 효과가 다릅니다. Resource Timing의 `nextHopProtocol`과 CDN log로 실제 h3 사용을 확인하고 TTFB, resource duration과 LCP를 h2 표본과 비교하겠습니다. 0-RTT는 replay 안전성을 검토하고 h2 fallback 경로도 유지해야 합니다.

## 장점

- TLS와 transport handshake를 결합해 재연결 비용을 낮출 여지가 있다.
- Stream별 손실 복구로 packet loss가 다른 resource stream에 미치는 영향을 줄일 수 있다.

## 단점

- 중간 network가 UDP를 제한하거나 CDN 설정이 준비되지 않으면 h3가 사용되지 않을 수 있다.
- 0-RTT와 connection migration 같은 이점은 요청 특성과 client 상황에 따라 제한된다.

## 주의사항 / 실무 팁

- LCP 이미지와 핵심 chunk를 중심으로 h3·h2의 TTFB, duration과 실제 LCP를 비교한다.
- UDP 제한과 h3 실패 뒤 h2 fallback 비율 및 연결 오류를 함께 관찰한다.
