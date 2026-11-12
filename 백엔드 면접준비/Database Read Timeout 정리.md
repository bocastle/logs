# Database Read Timeout 정리

## 핵심 요약

- 응답 없는 DB에 request thread가 무기한 묶이지 않는다.
- client 종료 뒤 server query가 계속 실행될 수 있다.
- server statement timeout을 client read timeout보다 짧게 둔다.

## 개념 설명

database read timeout은 connection이 이미 수립된 뒤 client socket이 DB response byte를 기다릴 수 있는 최대 시간이다.

socket timeout이 만료되면 driver가 read를 중단하고 connection을 폐기할 수 있지만 server query cancellation과 transaction rollback이 완료됐는지는 별도로 확인해야 한다.

## 예시

```text
connect timeout=3s -> connection established
socket read timeout=5s -> no response bytes for 5s
driver closes connection; server query cancellation 확인
```

read timeout은 느린 query, network stall, server overload를 모두 같은 client 증상으로 보일 수 있어 server query 상태와 network 지표를 함께 확인해야 한다.

## 면접 답변 예시

> Database read timeout은 connection이 만들어진 뒤 client가 response byte를 기다리는 socket 단계의 한도이며 connect와 pool checkout timeout과 다릅니다. Timeout이 나도 server query와 transaction이 즉시 취소됐다고 단정할 수 없어 query ID와 server 상태를 추적하겠습니다. 가능하면 server statement timeout을 client read timeout보다 짧게 두어 DB가 먼저 작업을 정리하게 합니다. Timeout connection은 pool에 돌려보내지 않고 retry는 멱등성과 남은 deadline을 확인해 DB 부하를 두 배로 만들지 않게 제한합니다.

## 장점

- client latency budget을 socket 계층에 적용한다.

## 단점

- 부분 response 중 timeout되면 결과 재사용이 불가능하다.

## 주의사항 / 실무 팁

- timeout connection은 pool에 반환하지 않는다.
