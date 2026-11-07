# Database Connection Max Lifetime 정리

## 핵심 요약

- network가 끊기 전에 connection을 계획적으로 교체한다.
- 너무 짧은 수명은 TLS·인증 handshake 부하를 높인다.
- 외부 idle·absolute timeout보다 여유 있게 짧게 둔다.

## 개념 설명

connection max lifetime은 pool connection을 생성 후 일정 시간이 지나면 폐기하고 새 connection으로 교체하는 수명 상한이다.

HikariCP maxLifetime을 DB·proxy·firewall의 강제 종료 시간보다 짧게 두고 작은 negative jitter로 connection recycling 시점을 분산한다.

## 예시

```text
network closes connections at 35m
maxLifetime=30m, per-connection negative jitter
retire only after borrowed connection returns
```

모든 connection이 같은 순간 생성되고 동일 수명에 도달하면 대량 reconnect가 생기므로 jitter와 minimum idle refill 속도가 중요하다.

## 면접 답변 예시

> Connection max lifetime은 pool connection이 DB나 proxy의 absolute timeout에 의해 갑자기 끊기기 전에 계획적으로 교체하는 수명 상한입니다. 외부 강제 종료보다 여유 있게 짧게 두되 너무 짧으면 TLS와 인증 handshake 부하가 커집니다. 같은 시점에 생성된 connection이 한꺼번에 만료되지 않도록 jitter와 refill 속도를 조절하겠습니다. Borrowed connection은 작업 중 강제로 닫지 않고 반환 뒤 retire하며 age histogram, create rate와 reset error를 함께 봅니다.

## 장점

- stale session state와 장기 connection 문제를 줄인다.

## 단점

- 동시 만료가 connection storm을 만든다.

## 주의사항 / 실무 팁

- connection age histogram과 create rate를 본다.
