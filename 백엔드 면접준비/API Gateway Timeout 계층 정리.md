# API Gateway Timeout 계층 정리

## 핵심 요약

- 어느 계층에서 제한 시간이 끝났는지 분류하기 쉽다.
- client timeout이 gateway보다 짧으면 client는 서버의 504를 받지 못한다.
- client, gateway, service, datastore 제한 시간을 route별 표로 관리한다.

## 개념 설명

API Gateway Timeout 계층은 client, gateway, service, datastore가 각자 제한 시간을 갖되 바깥에서 안쪽으로 짧아지도록 배치한 시간 예산 계층이다.

client 제한 시간 안에 gateway가 응답하고, service와 datastore는 더 짧은 deadline을 사용한다. 안쪽 계층이 실패하지 못하면 gateway는 504를 반환하고 취소를 전파한다.

## 예시

```text
client:    3000ms
gateway:   2800ms -> 504 on expiry
service:   2400ms
datastore: 1800ms
cleanup reserve: 600ms
```

datastore까지 제한 시간을 계층화해야 gateway 504 후에 query가 계속 돌며 connection pool을 점유하는 일을 막을 수 있다.

## 면접 답변 예시

> API timeout은 client, gateway, service와 datastore로 갈수록 더 짧아지게 두어 바깥 계층이 응답하고 정리할 시간을 남겨야 합니다. 독립적인 timer를 매 hop에서 새로 시작하기보다 absolute deadline과 남은 budget을 전달해 queue 대기와 retry도 같은 예산에 포함하겠습니다. 안쪽 timeout이 나면 query cancel과 request cancellation을 전파하되 driver가 실제로 작업을 멈추고 connection을 pool에 안전하게 반환했는지 확인합니다. 504 log에는 최초 deadline, 소진한 hop과 최종 원인을 남기고 client timeout보다 gateway가 늦어 응답 자체가 버려지는 경우를 부하 테스트합니다.

## 장점

- client 연결 종료 후 남는 service·datastore 작업을 줄인다.

## 단점

- datastore timeout이 길면 요청 취소 후에도 connection이 남는다.

## 주의사항 / 실무 팁

- 504 로그에 남은 deadline과 최종 upstream 구간을 남긴다.
