# Database Connection Timeout 정리

## 핵심 요약

- 죽은 DB endpoint에 요청 thread가 오래 묶이지 않는다.
- 너무 짧으면 일시 DNS·TLS 지연에도 연결이 실패한다.
- DNS·TCP·TLS 시간을 trace 단계로 나눈다.

## 개념 설명

database connection timeout은 client가 DB server와 network connection을 수립하는 단계에 적용하는 시간 한도다. DNS 해석, TCP connect와 TLS handshake 중 어느 구간까지 한 설정이 덮는지는 driver와 runtime 구현에 따라 다르다.

driver의 connect timeout은 socket read timeout이나 pool checkout timeout과 별개다. 일반적으로 host unreachable이나 SYN drop 같은 연결 수립 지연을 제한하며 TLS handshake까지 포함하는지는 해당 driver 문서를 확인해야 한다.

## 예시

```text
jdbc connectTimeout=3s
DNS=40ms TCP handshake=timeout -> connection attempt abort at 3s
socket read timeout=10s (query 응답용, 별도)
```

pool의 connectionTimeout은 기존 connection을 빌리는 대기 한도인 경우가 많아 새 network connection의 connect timeout과 이름만 보고 혼동하면 안 된다.

## 면접 답변 예시

> Database connection timeout은 새 network connection 수립 실패가 request thread를 오래 붙잡지 않게 하는 한도입니다. Pool checkout timeout, TCP connect와 TLS, query socket read timeout은 서로 다른 단계이며 이름이 비슷해도 같은 설정이 아닙니다. DNS가 driver connect timeout에 포함되는지도 구현마다 달라 resolver·TCP·TLS 시간을 trace에서 나눠 보겠습니다. Retry에는 backoff와 jitter를 두고 여러 instance가 동시에 재연결해 connection storm을 만들지 않게 제한합니다.

## 장점

- failover host 전환 시간을 제한한다.

## 단점

- 여러 instance 재연결이 동시에 일어나 connection storm이 된다.

## 주의사항 / 실무 팁

- connect retry에 exponential backoff와 jitter를 둔다.
