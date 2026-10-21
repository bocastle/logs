# Hinted Handoff 정리

## 핵심 요약

- 짧은 replica 장애에도 write availability를 유지한다.
- hint backlog가 holder disk를 채울 수 있다.
- hint age·count·delivery failure를 알림화한다.

## 개념 설명

hinted handoff는 target replica가 일시적으로 unavailable할 때 다른 node가 write와 대상 정보를 hint로 보관했다가 복구 후 전달하는 가용성 기법이다.

coordinator가 consistency level을 만족한 뒤 실패 replica용 hint를 TTL 동안 저장하고 background delivery로 해당 replica에 versioned mutation을 재적용한다.

## 예시

```text
write key=K replicas=[A,B,C], C down
A stores hint(target=C, key=K, version=17)
C recovers -> A delivers hint -> C applies version 17
```

hint는 전체 replica 수리를 대신하지 않으며 TTL을 넘긴 장애나 hint 손실은 anti-entropy repair가 처리해야 한다.

## 면접 답변 예시

> Hinted handoff는 target replica가 잠시 내려갔을 때 다른 node가 대상과 version을 hint로 보관했다가 복구 후 전달해 write availability를 유지하는 방식입니다. Hint 적용 시에도 현재 version과 conflict rule을 확인해 오래된 값이 새 값을 덮지 않게 해야 합니다. Hint TTL보다 장애가 길거나 holder가 hint를 잃으면 divergence가 남으므로 이 기능이 전체 repair를 대체하지는 않습니다. Hint age·backlog와 delivery failure를 관찰하고 장기 장애 뒤에는 anti-entropy repair를 실행합니다.

## 장점

- 복구 node가 누락 write를 빠르게 따라잡는다.

## 단점

- 오래된 hint가 새 값을 덮으면 데이터가 퇴행한다.

## 주의사항 / 실무 팁

- version conflict를 적용 시점에 검사한다.
