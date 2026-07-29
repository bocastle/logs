# 캐시 warm-up을 배포 흐름에 넣는 법 정리

## 핵심 요약

- traffic shift 전 cache miss 폭주를 줄일 수 있다.
- warm-up 작업 자체가 origin의 여유 용량을 소진할 수 있다.
- warm-up 부하에 별도 rate limit과 최대 동시성을 둔다.

## 개념 설명

배포 캐시 warm-up은 새 application version으로 traffic shift하기 전에 핵심 key를 버전된 cache namespace에 미리 채우고 전환 가능 여부를 판정하는 release 단계다.

배포 작업이 상위 key를 새 cache namespace에 채운 뒤 hit ratio와 origin QPS를 검증한다. error rate나 QPS가 abort threshold를 넘으면 traffic shift를 중단한다.

## 예시

```text
release=v42 cache namespace=product:v42
pre-warm top 10k keys at 200 keys/s
abort threshold: origin_qps>3000 or error_rate>1%
pass -> traffic shift 5% -> 25% -> 100%
```

cache namespace를 배포 버전과 분리해야 이전 schema의 value를 새 코드가 읽는 문제와 rollback 시 cache 충돌을 줄일 수 있다.

## 면접 답변 예시

> abort threshold로 warm-up이 원본 장애를 만드는 것을 막을 수 있다. 롤백 후 v42 warm-up이 계속되면 쓰지 않는 key와 원본 부하가 남는다. 롤백 시 warm-up worker를 취소하고 새 cache namespace의 TTL을 짧게 조정한다.

## 장점

- 새 version의 loader와 cache schema를 실제 원본 호출로 검증할 수 있다.

## 단점

- 잘못된 cache namespace를 채우면 배포 후에도 모두 miss가 난다.

## 주의사항 / 실무 팁

- abort threshold와 traffic shift 승인 조건을 release pipeline에 코드화한다.
