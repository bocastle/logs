# DB 커넥션 누수를 HikariCP 지표로 찾는 법 정리

## 핵심 요약

- 커넥션 누수는 빌린 DB 커넥션이 제때 반환되지 않아 풀의 여유 커넥션이 점점 줄어드는 문제다.
- HikariCP에서는 active, idle, pending, timeout, connection acquire time 같은 지표를 함께 보면 원인을 좁히기 좋다.
- 면접에서는 "풀 크기를 늘린다"보다 "누수인지, 긴 쿼리인지, 트랜잭션 경계 문제인지 구분한다"고 말하는 편이 실무적으로 들린다.

## 개념 설명

DB 커넥션 풀은 요청마다 커넥션을 새로 만들지 않고 미리 만들어 둔 커넥션을 빌려 쓰게 해 준다. 그런데 애플리케이션 코드가 커넥션을 반환하지 않거나, 트랜잭션이 오래 열린 채로 남아 있으면 풀은 금방 고갈된다.

이때 단순히 `maximumPoolSize`를 키우면 잠깐은 좋아 보일 수 있다. 하지만 누수가 원인이라면 더 큰 풀도 결국 찬다. 그래서 먼저 지표를 봐야 한다. active가 계속 높고 idle이 거의 없으며 pending이 늘어난다면 풀을 기다리는 요청이 많다는 뜻이다. 동시에 DB 쪽에서 긴 쿼리나 idle in transaction이 있는지도 같이 봐야 한다.

HikariCP에는 leak detection threshold 같은 보조 장치도 있다. 다만 이 값은 운영에서 항상 켜 두는 만능 해결책이 아니라, 의심 구간을 좁힐 때 로그 신호로 활용하는 편이 좋다.

## 예시

```text
hikaricp.connections.active   20
hikaricp.connections.idle      0
hikaricp.connections.pending   14
hikaricp.connections.timeout   증가 중

DB 확인:
- 오래 열린 transaction
- idle in transaction
- 느린 query
- 커넥션을 닫지 않는 코드 경로
```

이런 상황이면 "트래픽이 많다"만으로 끝내지 말고, 특정 API 이후 active가 내려오지 않는지, timeout이 발생하기 직전에 어떤 쿼리가 많았는지 같이 봐야 한다.

## 면접 답변 예시

> DB 커넥션 누수를 의심할 때는 먼저 풀의 active, idle, pending, timeout 지표를 봅니다. active가 계속 높고 idle이 없고 pending이 쌓이면 요청들이 커넥션을 기다리는 상태라고 볼 수 있습니다. 그다음 DB에서 긴 쿼리나 idle in transaction을 확인하고, 애플리케이션에서는 트랜잭션 경계와 커넥션 반환 누락을 봅니다. 풀 크기를 바로 키우기보다 누수인지 부하인지 구분하고, 필요하면 HikariCP의 leak detection 로그로 어느 코드 경로에서 오래 잡고 있는지 좁히겠습니다.

## 장점

- 장애 원인을 풀 크기 부족, 느린 쿼리, 커넥션 누수로 나눠서 볼 수 있다.
- 지표 기반으로 설명하면 면접 답변이 추측처럼 들리지 않는다.
- 운영 알림 기준을 만들기 쉽다.

## 단점

- 지표 하나만 보면 오판하기 쉽다. active가 높다고 항상 누수는 아니다.
- leak detection threshold를 너무 낮게 잡으면 정상적인 긴 작업도 누수처럼 보일 수 있다.
- DB 지표와 애플리케이션 지표를 같이 보지 않으면 원인을 한쪽으로 몰기 쉽다.

## 주의사항 / 실무 팁

- `try-with-resources`, 트랜잭션 종료, 예외 경로에서의 반환 누락을 먼저 확인한다.
- 커넥션 풀 timeout은 사용자 장애로 바로 이어지므로 pending과 timeout 증가를 알림으로 잡는 것이 좋다.
- 배치 작업이나 리포트 쿼리처럼 오래 걸리는 작업은 일반 API와 풀을 분리하는 것도 고려할 수 있다.
