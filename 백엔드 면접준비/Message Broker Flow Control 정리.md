# Message Broker Flow Control 정리

## 핵심 요약

- broker의 memory와 disk 고갈을 늦출 수 있다.
- producer 입장에서는 갑작스러운 timeout과 장애처럼 보인다.
- flow control active 시간과 publish latency를 함께 본다.

## 개념 설명

Message Broker Flow Control은 broker의 memory, disk, consumer lag가 위험할 때 producer publish 속도를 늦추거나 막는 보호 장치다.

broker가 credit, block, throttle 신호를 보내면 producer는 publish latency 증가나 timeout으로 체감한다. 원인 지표와 함께 해석해야 한다.

## 예시

```text
broker memory > 80%
or disk free < 10%
-> flow control active
producer publish_latency p95=2s
```

flow control은 broker가 더 큰 장애로 가기 전 보내는 압력 신호다. 애플리케이션 버그로만 보면 원인을 놓친다.

## 면접 답변 예시

> Message broker flow control은 memory나 disk가 위험한 수준이거나 consumer가 따라오지 못할 때 producer 속도를 제한하는 backpressure입니다. Producer에서는 publish latency 증가나 timeout처럼 보이므로 application 오류로만 보지 않고 broker의 watermark, disk free와 consumer lag를 같은 시각에 확인하겠습니다. Ack를 받기 전에 timeout이 나면 저장 여부를 모른 채 retry해 중복이 생길 수 있어 message ID와 멱등 consumer를 준비합니다. Flow control 활성 시간과 publish reject·timeout을 경보로 묶고 근본 원인인 느린 consumer나 용량 부족을 먼저 해결합니다.

## 장점

- producer가 downstream 지연을 신호로 받아 속도를 낮춘다.

## 단점

- 근본 원인인 consumer lag나 disk 부족을 해결하지 않으면 반복된다.

## 주의사항 / 실무 팁

- consumer lag, disk free, memory watermark를 원인 지표로 묶는다.
