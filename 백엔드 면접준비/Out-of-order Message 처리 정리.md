# Out-of-order Message 처리 정리

## 핵심 요약

- 네트워크 지연과 재시도 중에도 상태 역전을 막을 수 있다.
- sequence 발급 기준이 흔들리면 consumer 판단도 틀린다.
- 메시지 schema에 aggregate_id와 sequence를 명시한다.

## 개념 설명

Out-of-order Message 처리는 이벤트가 생성 순서와 다른 순서로 도착할 때 상태를 잘못 되돌리지 않게 하는 소비자 설계다.

aggregate별 version이나 sequence를 메시지에 넣고, consumer는 마지막 처리 version보다 오래된 메시지를 무시하거나 보류한다.

## 예시

```text
received order-7 seq=12 -> apply
received order-7 seq=11 -> ignore or park
received order-7 seq=13 -> apply
```

순서 문제를 broker 전체 순서로 풀려고 하면 처리량이 줄어든다. 필요한 key 범위에서 version을 검증하는 편이 현실적이다.

## 면접 답변 예시

> Out-of-order message 처리는 늦게 도착한 event가 최신 상태를 과거로 되돌리지 않게 하는 설계입니다. Message에 `aggregate_id`와 단조 증가하는 sequence를 넣고 consumer가 마지막 처리 sequence와 비교해 오래된 것은 무시하겠습니다. 다음 번호보다 큰 event가 먼저 오면 업무 규칙에 따라 보류하거나 현재 상태만으로 안전하게 적용할지 정해야 하며, broker 전체가 아니라 aggregate key 범위에서 순서를 다루는 편이 현실적입니다. 마지막 sequence는 durable store에 저장하고 gap age와 out-of-order count를 관찰하되, 이미 실행된 외부 side effect는 단순 무시로 취소되지 않는다는 점도 별도로 처리합니다.

## 장점

- aggregate 단위 순서 보장으로 처리량과 정합성을 균형 있게 잡는다.

## 단점

- gap을 오래 보류하면 메모리나 보관 비용이 커진다.

## 주의사항 / 실무 팁

- 마지막 처리 sequence를 durable store에 저장한다.
