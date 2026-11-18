# Cache Aside Write Race 정리

## 핵심 요약

- 읽기 경로는 단순하게 cache miss 때 DB를 채우면 된다.
- 동시 read가 오래된 값을 다시 캐시에 넣을 수 있다.
- cache value에 DB version이나 updated_at을 포함한다.

## 개념 설명

Cache Aside Write Race는 DB 갱신과 캐시 삭제·재생성 사이에 경쟁이 생겨 오래된 값이 캐시에 다시 들어가는 문제다.

write 후 cache delete, delayed double delete, versioned value, read repair 같은 방식으로 DB version보다 낮은 cache value를 줄인다.

## 예시

```text
T1 read old DB -> pause
T2 update DB version=8 -> DEL cache
T1 SET cache version=7
reader detects version < DB hint -> refresh
```

cache aside는 단순하지만 쓰기 경쟁에서 stale value가 되살아날 수 있다. version 정보가 있으면 감지가 쉬워진다.

## 면접 답변 예시

> Cache aside write race는 reader가 예전 DB 값을 읽은 뒤 멈춘 사이 writer가 새 값을 commit하고 cache를 지웠는데, 늦게 깨어난 reader가 예전 값을 다시 cache에 넣는 상황입니다. 기본적으로 DB commit 뒤 cache를 삭제하고 삭제 실패를 retry하지만, delayed double delete는 경쟁 구간을 줄이는 완화책이지 완전한 보장은 아닙니다. 더 강한 정합성이 필요하면 version을 포함한 conditional write, versioned key나 outbox·CDC 기반 invalidation처럼 순서를 검증할 수 있는 방식을 선택하겠습니다. TTL만 믿지 않고 invalidation failure, stale read와 cache·DB version 차이를 관측합니다.

## 장점

- 쓰기 후 삭제 전략으로 많은 서비스에서 쉽게 적용할 수 있다.

## 단점

- double delete는 timing에 의존해 완전한 보장은 아니다.

## 주의사항 / 실무 팁

- write 후 delete 실패를 retry 가능한 작업으로 남긴다.
